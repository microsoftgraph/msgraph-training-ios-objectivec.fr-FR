<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Pour ce faire, vous allez intégrer la [bibliothèque d’authentification Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) dans l’application.

1. Créez un nouveau fichier de **liste de propriétés** dans le projet **GraphTutorial** nommé **AuthSettings. plist**.
1. Ajoutez les éléments suivants au fichier dans le dictionnaire **racine** .

    | Clé | Type | Valeur |
    |-----|------|-------|
    | `AppId` | String | ID de l’application à partir du portail Azure |
    | `GraphScopes` | Tableau | Deux valeurs de chaîne `User.Read` : et`Calendars.Read` |

    ![Capture d’écran du fichier AuthSettings. plist dans Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **AuthSettings. plist** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application.

## <a name="implement-sign-in"></a>Implémentation de la connexion

Dans cette section, vous allez configurer le projet pour MSAL, créer une classe de gestionnaire d’authentification, puis mettre à jour l’application pour se connecter et se déconnecter.

### <a name="configure-project-for-msal"></a>Configurer Project pour MSAL

1. Ajoutez un nouveau groupe de trousseau aux fonctionnalités de votre projet.
    1. Sélectionnez le projet **GraphTutorial** , puis **signez les fonctionnalités de &**.
    1. Sélectionnez **+ capacité**, puis double-cliquez sur **partage de trousseau**.
    1. Ajoutez un groupe de trousseau avec la valeur `com.microsoft.adalcache`.

1. Contrôle cliquez sur **info. plist** , sélectionnez **ouvrir en tant que**, puis **code source**.
1. Ajoutez les éléments suivants à `<dict>` l’intérieur de l’élément.

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

1. Ouvrez **AppDelegate. m** et ajoutez l’instruction import suivante en haut du fichier.

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. Ajoutez la fonction suivante à la classe `AppDelegate`.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Créer le gestionnaire d’authentification

1. Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **AuthenticationManager**. Choisissez **NSObject** dans la sous- **classe du** champ.
1. Ouvrez **AuthenticationManager. h** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. Ouvrez **AuthenticationManager. m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Ajouter la connexion et la déconnexion

1. Ouvrez le fichier **SignInViewController. m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. Ouvrez **WelcomeViewController. m** et ajoutez l’instruction `import` suivante en haut du fichier.

    ```objc
    #import "AuthenticationManager.h"
    ```

1. Remplacez la fonction `signOut` existante par ce qui suit.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. Enregistrez vos modifications et redémarrez l’application dans Simulator.

Si vous vous connectez à l’application, vous devriez voir un jeton d’accès affiché dans la fenêtre sortie dans Xcode.

![Capture d’écran de la fenêtre sortie dans Xcode affichant un jeton d’accès](./images/access-token-output.png)

## <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Dans cette section, vous allez créer une classe d’assistance pour contenir tous les appels à Microsoft Graph et mettre à `WelcomeViewController` jour le pour utiliser cette nouvelle classe afin d’obtenir l’utilisateur connecté.

1. Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **GraphManager**. Choisissez **NSObject** dans la sous- **classe du** champ.
1. Ouvrez **GraphManager. h** et remplacez son contenu par le code suivant.

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. Ouvrez **GraphManager. m** et remplacez son contenu par le code suivant.

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

    - (void) getMeWithCompletionBlock:(GetMeCompletionBlock)completionBlock {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me", MSGraphBaseURL];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completionBlock(nil, graphError);
                } else {
                    completionBlock(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. Ouvrez **WelcomeViewController. m** et ajoutez les instructions `#import` suivantes en haut du fichier.

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. Ajoutez la propriété suivante à la `WelcomeViewController` déclaration de l’interface.

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. Remplacez le code `viewDidLoad` existant par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

Si vous enregistrez vos modifications et redémarrez l’application maintenant, une fois connecté, l’interface utilisateur est mise à jour avec le nom d’affichage et l’adresse de messagerie de l’utilisateur.
