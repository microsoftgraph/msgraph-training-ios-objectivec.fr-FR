<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Pour ce faire, vous allez intégrer la [bibliothèque d’authentification Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) dans l’application.

1. Créez un nouveau fichier de **liste de propriétés** dans le projet **GraphTutorial** nommé **AuthSettings. plist**.
1. Ajoutez les éléments suivants au fichier dans le dictionnaire **racine** .

    | Key | Type | Valeur |
    |-----|------|-------|
    | `AppId` | String | ID de l’application à partir du portail Azure |
    | `GraphScopes` | Tableau | Deux valeurs de chaîne `User.Read` : et`Calendars.Read` |

    ![Capture d’écran du fichier AuthSettings. plist dans Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **AuthSettings. plist** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application.

## <a name="implement-sign-in"></a>Mettre en œuvre la connexion

Dans cette section, vous allez configurer le projet pour MSAL, créer une classe de gestionnaire d’authentification, puis mettre à jour l’application pour se connecter et se déconnecter.

### <a name="configure-project-for-msal"></a>Configurer Project pour MSAL

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

    ```objc
    - (BOOL)application:(UIApplication *)app
                openURL:(NSURL *)url
                options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url
                                             sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```

### <a name="create-authentication-manager"></a>Créer le gestionnaire d’authentification

1. Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **AuthenticationManager**. Choisissez **NSObject** dans la sous- **classe du** champ.
1. Ouvrez **AuthenticationManager. h** et remplacez son contenu par le code suivant.

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSAL/MSAL.h>

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetTokenCompletionBlock)(NSString* _Nullable accessToken, NSError* _Nullable error);

    @interface AuthenticationManager : NSObject

    + (id) instance;
    - (void) getTokenInteractivelyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) getTokenSilentlyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) signOut;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. Ouvrez **AuthenticationManager. m** et remplacez son contenu par le code suivant.

    ```objc
    #import "AuthenticationManager.h"

    @interface AuthenticationManager()

    @property NSString* appId;
    @property NSArray<NSString*>* graphScopes;
    @property MSALPublicClientApplication* publicClient;

    @end

    @implementation AuthenticationManager

    + (id) instance {
        static AuthenticationManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Get app ID and scopes from AuthSettings.plist
            NSString* authConfigPath =
            [NSBundle.mainBundle pathForResource:@"AuthSettings" ofType:@"plist"];
            NSString* bundleId = NSBundle.mainBundle.bundleIdentifier;
            NSDictionary* authConfig = [NSDictionary dictionaryWithContentsOfFile:authConfigPath];

            self.appId = authConfig[@"AppId"];
            self.graphScopes = authConfig[@"GraphScopes"];

            // Create the MSAL client
            self.publicClient = [[MSALPublicClientApplication alloc] initWithClientId:self.appId
                                                                        keychainGroup:bundleId
                                                                                error:nil];
        }

        return self;
    }

    - (void) getTokenInteractivelyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Call acquireToken to open a browser so the user can sign in
        [self.publicClient
         acquireTokenForScopes:self.graphScopes
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {

            // Check error
            if (error) {
                completionBlock(nil, error);
                return;
            }

            // Check result
            if (!result) {
                NSMutableDictionary* details = [NSMutableDictionary dictionary];
                [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                return;
            }

            NSLog(@"Got token interactively: %@", result.accessToken);
            completionBlock(result.accessToken, nil);
        }];
    }

    - (void) getTokenSilentlyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Check if there is an account in the cache
        NSError* msalError;
        MSALAccount* account = [self.publicClient allAccounts:&msalError].firstObject;

        if (msalError || !account) {
            NSMutableDictionary* details = [NSMutableDictionary dictionary];
            [details setValue:@"Could not retrieve account from cache" forKey:NSDebugDescriptionErrorKey];
            completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
            return;
        }

        // Attempt to get token silently
        [self.publicClient
         acquireTokenSilentForScopes:self.graphScopes account:account
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {
             // Check error
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // Check result
             if (!result) {
                 NSMutableDictionary* details = [NSMutableDictionary dictionary];
                 [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                 completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                 return;
             }

             NSLog(@"Got token silently: %@", result.accessToken);
             completionBlock(result.accessToken, nil);
         }];
    }

    - (void) signOut {
        NSError* msalError;
        NSArray* accounts = [self.publicClient allAccounts:&msalError];

        if (msalError) {
            NSLog(@"Error getting accounts from cache: %@", msalError.debugDescription);
            return;
        }

        for (id account in accounts) {
            [self.publicClient removeAccount:account error:nil];
        }
    }

    @end
    ```

### <a name="add-sign-in-and-sign-out"></a>Ajouter la connexion et la déconnexion

1. Ouvrez le fichier **SignInViewController. m** et remplacez son contenu par le code suivant.

    ```objc
    #import "SignInViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"

    @interface SignInViewController ()
    @property SpinnerViewController* spinner;
    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenSilentlyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // If there is no token or if there's an error,
                     // no user is signed in, so stay here
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
        }];
    }

    - (IBAction)signIn {
        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenInteractivelyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // Show the error and stay on the sign-in page
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error signing in"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
         }];
    }
    @end
    ```

1. Ouvrez **WelcomeViewController. m** et remplacez la fonction `signOut` existante par ce qui suit.

    ```objc
    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }
    ```

1. Enregistrez vos modifications et redémarrez l’application dans Simulator.

Si vous vous connectez à l’application, vous devriez voir un jeton d’accès affiché dans la fenêtre sortie dans Xcode.

![Capture d’écran de la fenêtre sortie dans Xcode affichant un jeton d’accès](./images/access-token-output.png)

## <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Dans cette section, vous allez créer une classe d’assistance pour contenir tous les appels à Microsoft Graph et mettre à `WelcomeViewController` jour le pour utiliser cette nouvelle classe afin d’obtenir l’utilisateur connecté.

1. Ouvrez **AuthenticationManager. h** et ajoutez l’instruction `#import` suivante en haut du fichier.

    ```objc
    #import <MSGraphMSALAuthProvider/MSGraphMSALAuthProvider.h>
    ```

1. Ajoutez la ligne suivante à l' `@interface` intérieur de la déclaration.

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider;
    ```

1. Ouvrez **AuthenticationManager. m** et ajoutez la fonction suivante à la `AuthenticationManager` classe.

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider {
        // Create an MSAL auth provider for use with the Graph client
        return [[MSALAuthenticationProvider alloc]
                initWithPublicClientApplication:self.publicClient
                andScopes:self.graphScopes];
    }
    ```

1. Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **GraphManager**. Choisissez **NSObject** dans la sous- **classe du** champ.
1. Ouvrez **GraphManager. h** et remplacez son contenu par le code suivant.

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
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

            MSALAuthenticationProvider* authProvider =
            [AuthenticationManager.instance getGraphAuthProvider];

            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:authProvider];
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

    ```objc
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
                                                message:error.debugDescription
                                                preferredStyle:UIAlertControllerStyleAlert];

                    UIAlertAction* okButton = [UIAlertAction
                                               actionWithTitle:@"OK"
                                               style:UIAlertActionStyleDefault
                                               handler:nil];

                    [alert addAction:okButton];
                    [self presentViewController:alert animated:true completion:nil];
                    return;
                }

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }
    ```

Si vous enregistrez vos modifications et redémarrez l’application maintenant, une fois connecté, l’interface utilisateur est mise à jour avec le nom d’affichage et l’adresse de messagerie de l’utilisateur.
