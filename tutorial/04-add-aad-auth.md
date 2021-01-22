<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cette étape est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Pour ce faire, vous allez intégrer la bibliothèque d’authentification [Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) dans l’application.

1. Créez un **fichier de liste de** propriétés dans le projet **GraphTutorial** nommé **AuthSettings.plist**.
1. Ajoutez les éléments suivants au fichier dans **le dictionnaire** racine.

    | Clé | Type | Valeur |
    |-----|------|-------|
    | `AppId` | Chaîne | ID d’application à partir du portail Azure |
    | `GraphScopes` | Tableau | Trois valeurs de `User.Read` chaîne : `MailboxSettings.Read` , et `Calendars.ReadWrite` |

    ![Capture d’écran du fichier AuthSettings.plist dans Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier **AuthSettings.plist** du contrôle source afin d’éviter toute fuite accidentelle de votre ID d’application.

## <a name="implement-sign-in"></a>Implémentation de la connexion

Dans cette section, vous allez configurer le projet pour MSAL, créer une classe de gestionnaire d’authentification et mettre à jour l’application pour qu’elle se connecte et se connecte.

### <a name="configure-project-for-msal"></a>Configurer le projet pour MSAL

1. Ajoutez un nouveau groupe dechains aux fonctionnalités de votre projet.
    1. Sélectionnez **le projet GraphTutorial,** puis & **fonctionnalités de signature.**
    1. Sélectionnez **+ Fonctionnalité,** puis double-cliquez sur **Partage duchain.**
    1. Ajoutez un groupe de chaînes de clés avec la valeur `com.microsoft.adalcache` .

1. Contrôlez **cliquez sur Info.plist** et **sélectionnez Ouvrir sous,** puis **Code source.**
1. Ajoutez ce qui suit à l’intérieur de `<dict>` l’élément.

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. Ouvrez **AppDelegate.m** et ajoutez l’instruction import suivante en haut du fichier.

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. Ajoutez la fonction suivante à la classe `AppDelegate`.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Créer un gestionnaire d’authentification

1. Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **AuthenticationManager**. Choisissez **NSObject dans** la **sous-classe du** champ.
1. Ouvrez **AuthenticationManager.h** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. Ouvrez **AuthenticationManager.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Ajout de la signature et de la sortie

1. Ouvrez **le fichier SignInViewController.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. Ouvrez **WelcomeViewController.m** et ajoutez l’instruction `import` suivante en haut du fichier.

    ```objc
    #import "AuthenticationManager.h"
    ```

1. Remplacez la fonction `signOut` existante par ce qui suit.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. Enregistrez vos modifications et redémarrez l’application dans simulateur.

Si vous vous connectez à l’application, un jeton d’accès doit s’afficher dans la fenêtre de sortie dans Xcode.

![Capture d’écran de la fenêtre de sortie dans Xcode affichant un jeton d’accès](./images/access-token-output.png)

## <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Dans cette section, vous allez créer une classe d’aide pour contenir tous les appels à Microsoft Graph et mettre à jour la classe pour utiliser cette nouvelle classe afin d’obtenir l’utilisateur `WelcomeViewController` connecté.

1. Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **GraphManager**. Choisissez **NSObject dans** la **sous-classe du** champ.
1. Ouvrez **GraphManager.h** et remplacez son contenu par le code suivant.

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user,
                                         NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. Ouvrez **GraphManager.m** et remplacez son contenu par le code suivant.

    ```objc
    #import "GraphManager.h"

    @interface GraphManager()

    @property MSHTTPClient* graphClient;

    @end

    @implementation GraphManager

    + (id) instance {
        static GraphManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:AuthenticationManager.instance];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me?%@",
                                 MSGraphBaseURL,
                                 @"$select=displayName,mail,mailboxSettings,userPrincipalName"];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completion(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completion(nil, graphError);
                } else {
                    completion(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. Ouvrez **WelcomeViewController.m** et ajoutez les `#import` instructions suivantes en haut du fichier.

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. Ajoutez la propriété suivante à la déclaration `WelcomeViewController` d’interface.

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. Remplacez `viewDidLoad` l’existant par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

Si vous enregistrez vos modifications et redémarrez l’application maintenant, après la mise à jour de l’interface utilisateur avec le nom d’affichage et l’adresse e-mail de l’utilisateur.
