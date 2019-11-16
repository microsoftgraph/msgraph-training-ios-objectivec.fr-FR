<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1bd02-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="1bd02-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="1bd02-102">Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1bd02-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="1bd02-103">Pour ce faire, vous allez intégrer la [bibliothèque d’authentification Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="1bd02-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="1bd02-104">Créez un nouveau fichier de **liste de propriétés** dans le projet **GraphTutorial** nommé **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="1bd02-105">Ajoutez les éléments suivants au fichier dans le dictionnaire **racine** .</span><span class="sxs-lookup"><span data-stu-id="1bd02-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="1bd02-106">Clé</span><span class="sxs-lookup"><span data-stu-id="1bd02-106">Key</span></span> | <span data-ttu-id="1bd02-107">Type</span><span class="sxs-lookup"><span data-stu-id="1bd02-107">Type</span></span> | <span data-ttu-id="1bd02-108">Valeur</span><span class="sxs-lookup"><span data-stu-id="1bd02-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="1bd02-109">String</span><span class="sxs-lookup"><span data-stu-id="1bd02-109">String</span></span> | <span data-ttu-id="1bd02-110">ID de l’application à partir du portail Azure</span><span class="sxs-lookup"><span data-stu-id="1bd02-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="1bd02-111">Tableau</span><span class="sxs-lookup"><span data-stu-id="1bd02-111">Array</span></span> | <span data-ttu-id="1bd02-112">Deux valeurs de chaîne `User.Read` : et`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="1bd02-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Capture d’écran du fichier AuthSettings. plist dans Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="1bd02-114">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **AuthSettings. plist** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application.</span><span class="sxs-lookup"><span data-stu-id="1bd02-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="1bd02-115">Mettre en œuvre la connexion</span><span class="sxs-lookup"><span data-stu-id="1bd02-115">Implement sign-in</span></span>

<span data-ttu-id="1bd02-116">Dans cette section, vous allez configurer le projet pour MSAL, créer une classe de gestionnaire d’authentification, puis mettre à jour l’application pour se connecter et se déconnecter.</span><span class="sxs-lookup"><span data-stu-id="1bd02-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="1bd02-117">Configurer Project pour MSAL</span><span class="sxs-lookup"><span data-stu-id="1bd02-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="1bd02-118">Ajoutez un nouveau groupe de trousseau aux fonctionnalités de votre projet.</span><span class="sxs-lookup"><span data-stu-id="1bd02-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="1bd02-119">Sélectionnez le projet **GraphTutorial** , puis **signez les fonctionnalités de &**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="1bd02-120">Sélectionnez **+ capacité**, puis double-cliquez sur **partage de trousseau**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="1bd02-121">Ajoutez un groupe de trousseau avec la valeur `com.microsoft.adalcache`.</span><span class="sxs-lookup"><span data-stu-id="1bd02-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="1bd02-122">Contrôle cliquez sur **info. plist** , sélectionnez **ouvrir en tant que**, puis **code source**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="1bd02-123">Ajoutez les éléments suivants à `<dict>` l’intérieur de l’élément.</span><span class="sxs-lookup"><span data-stu-id="1bd02-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="1bd02-124">Ouvrez **AppDelegate. m** et ajoutez l’instruction import suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="1bd02-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="1bd02-125">Ajoutez la fonction suivante à la classe `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="1bd02-125">Add the following function to the `AppDelegate` class.</span></span>

    ```objc
    - (BOOL)application:(UIApplication *)app
                openURL:(NSURL *)url
                options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url
                                             sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="1bd02-126">Créer le gestionnaire d’authentification</span><span class="sxs-lookup"><span data-stu-id="1bd02-126">Create authentication manager</span></span>

1. <span data-ttu-id="1bd02-127">Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="1bd02-128">Choisissez **NSObject** dans la sous- **classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="1bd02-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="1bd02-129">Ouvrez **AuthenticationManager. h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSAL/MSAL.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetTokenCompletionBlock)(NSString* _Nullable accessToken, NSError* _Nullable error);

    @interface AuthenticationManager : NSObject<MSAuthenticationProvider>

    + (id) instance;
    - (void) getTokenInteractivelyWithParentView: (UIViewController*) parentView andCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) getTokenSilentlyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) signOut;
    - (void) getAccessTokenForProviderOptions:(id<MSAuthenticationProviderOptions>)authProviderOptions andCompletion:(void (^)(NSString *, NSError *))completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="1bd02-130">Ouvrez **AuthenticationManager. m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

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
            NSDictionary* authConfig = [NSDictionary dictionaryWithContentsOfFile:authConfigPath];

            self.appId = authConfig[@"AppId"];
            self.graphScopes = authConfig[@"GraphScopes"];

            // Create the MSAL client
            self.publicClient = [[MSALPublicClientApplication alloc] initWithClientId:self.appId error:nil];
        }

        return self;
    }

    - (void) getAccessTokenForProviderOptions:(id<MSAuthenticationProviderOptions>)authProviderOptions andCompletion:(void (^)(NSString * _Nonnull, NSError * _Nonnull))completion {
        [self getTokenSilentlyWithCompletionBlock:completion];
    }

    - (void) getTokenInteractivelyWithParentView:(UIViewController *)parentView andCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        MSALWebviewParameters* webParameters = [[MSALWebviewParameters alloc] initWithParentViewController:parentView];
        MSALInteractiveTokenParameters* interactiveParameters =
        [[MSALInteractiveTokenParameters alloc]initWithScopes:self.graphScopes webviewParameters:webParameters];

        // Call acquireToken to open a browser so the user can sign in
        [self.publicClient
         acquireTokenWithParameters:interactiveParameters
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

        MSALSilentTokenParameters* silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:self.graphScopes
                                                                                                account:account];

        // Attempt to get token silently
        [self.publicClient
         acquireTokenSilentWithParameters:silentParameters
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

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="1bd02-131">Ajouter la connexion et la déconnexion</span><span class="sxs-lookup"><span data-stu-id="1bd02-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="1bd02-132">Ouvrez le fichier **SignInViewController. m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

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
         getTokenInteractivelyWithParentView:self
         andCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
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

1. <span data-ttu-id="1bd02-133">Ouvrez **WelcomeViewController. m** et remplacez la fonction `signOut` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="1bd02-133">Open **WelcomeViewController.m** and replace the existing `signOut` function with the following.</span></span>

    ```objc
    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }
    ```

1. <span data-ttu-id="1bd02-134">Enregistrez vos modifications et redémarrez l’application dans Simulator.</span><span class="sxs-lookup"><span data-stu-id="1bd02-134">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="1bd02-135">Si vous vous connectez à l’application, vous devriez voir un jeton d’accès affiché dans la fenêtre sortie dans Xcode.</span><span class="sxs-lookup"><span data-stu-id="1bd02-135">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Capture d’écran de la fenêtre sortie dans Xcode affichant un jeton d’accès](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="1bd02-137">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="1bd02-137">Get user details</span></span>

<span data-ttu-id="1bd02-138">Dans cette section, vous allez créer une classe d’assistance pour contenir tous les appels à Microsoft Graph et mettre à `WelcomeViewController` jour le pour utiliser cette nouvelle classe afin d’obtenir l’utilisateur connecté.</span><span class="sxs-lookup"><span data-stu-id="1bd02-138">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="1bd02-139">Créez une **classe Touch de cacao** dans le projet **GraphTutorial** nommé **GraphManager**.</span><span class="sxs-lookup"><span data-stu-id="1bd02-139">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="1bd02-140">Choisissez **NSObject** dans la sous- **classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="1bd02-140">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="1bd02-141">Ouvrez **GraphManager. h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-141">Open **GraphManager.h** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="1bd02-142">Ouvrez **GraphManager. m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-142">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="1bd02-143">Ouvrez **WelcomeViewController. m** et ajoutez les instructions `#import` suivantes en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="1bd02-143">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="1bd02-144">Ajoutez la propriété suivante à la `WelcomeViewController` déclaration de l’interface.</span><span class="sxs-lookup"><span data-stu-id="1bd02-144">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="1bd02-145">Remplacez le code `viewDidLoad` existant par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1bd02-145">Replace the existing `viewDidLoad` with the following code.</span></span>

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

<span data-ttu-id="1bd02-146">Si vous enregistrez vos modifications et redémarrez l’application maintenant, une fois connecté, l’interface utilisateur est mise à jour avec le nom d’affichage et l’adresse de messagerie de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="1bd02-146">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
