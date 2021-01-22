<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="50873-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="50873-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="50873-102">Cette étape est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="50873-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="50873-103">Pour ce faire, vous allez intégrer la bibliothèque d’authentification [Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="50873-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="50873-104">Créez un **fichier de liste de** propriétés dans le projet **GraphTutorial** nommé **AuthSettings.plist**.</span><span class="sxs-lookup"><span data-stu-id="50873-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="50873-105">Ajoutez les éléments suivants au fichier dans **le dictionnaire** racine.</span><span class="sxs-lookup"><span data-stu-id="50873-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="50873-106">Clé</span><span class="sxs-lookup"><span data-stu-id="50873-106">Key</span></span> | <span data-ttu-id="50873-107">Type</span><span class="sxs-lookup"><span data-stu-id="50873-107">Type</span></span> | <span data-ttu-id="50873-108">Valeur</span><span class="sxs-lookup"><span data-stu-id="50873-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="50873-109">Chaîne</span><span class="sxs-lookup"><span data-stu-id="50873-109">String</span></span> | <span data-ttu-id="50873-110">ID d’application à partir du portail Azure</span><span class="sxs-lookup"><span data-stu-id="50873-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="50873-111">Tableau</span><span class="sxs-lookup"><span data-stu-id="50873-111">Array</span></span> | <span data-ttu-id="50873-112">Trois valeurs de `User.Read` chaîne : `MailboxSettings.Read` , et `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="50873-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Capture d’écran du fichier AuthSettings.plist dans Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="50873-114">Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier **AuthSettings.plist** du contrôle source afin d’éviter toute fuite accidentelle de votre ID d’application.</span><span class="sxs-lookup"><span data-stu-id="50873-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="50873-115">Implémentation de la connexion</span><span class="sxs-lookup"><span data-stu-id="50873-115">Implement sign-in</span></span>

<span data-ttu-id="50873-116">Dans cette section, vous allez configurer le projet pour MSAL, créer une classe de gestionnaire d’authentification et mettre à jour l’application pour qu’elle se connecte et se connecte.</span><span class="sxs-lookup"><span data-stu-id="50873-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="50873-117">Configurer le projet pour MSAL</span><span class="sxs-lookup"><span data-stu-id="50873-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="50873-118">Ajoutez un nouveau groupe dechains aux fonctionnalités de votre projet.</span><span class="sxs-lookup"><span data-stu-id="50873-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="50873-119">Sélectionnez **le projet GraphTutorial,** puis & **fonctionnalités de signature.**</span><span class="sxs-lookup"><span data-stu-id="50873-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="50873-120">Sélectionnez **+ Fonctionnalité,** puis double-cliquez sur **Partage duchain.**</span><span class="sxs-lookup"><span data-stu-id="50873-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="50873-121">Ajoutez un groupe de chaînes de clés avec la valeur `com.microsoft.adalcache` .</span><span class="sxs-lookup"><span data-stu-id="50873-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="50873-122">Contrôlez **cliquez sur Info.plist** et **sélectionnez Ouvrir sous,** puis **Code source.**</span><span class="sxs-lookup"><span data-stu-id="50873-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="50873-123">Ajoutez ce qui suit à l’intérieur de `<dict>` l’élément.</span><span class="sxs-lookup"><span data-stu-id="50873-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="50873-124">Ouvrez **AppDelegate.m** et ajoutez l’instruction import suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="50873-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="50873-125">Ajoutez la fonction suivante à la classe `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="50873-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="50873-126">Créer un gestionnaire d’authentification</span><span class="sxs-lookup"><span data-stu-id="50873-126">Create authentication manager</span></span>

1. <span data-ttu-id="50873-127">Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="50873-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="50873-128">Choisissez **NSObject dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="50873-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="50873-129">Ouvrez **AuthenticationManager.h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. <span data-ttu-id="50873-130">Ouvrez **AuthenticationManager.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="50873-131">Ajout de la signature et de la sortie</span><span class="sxs-lookup"><span data-stu-id="50873-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="50873-132">Ouvrez **le fichier SignInViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. <span data-ttu-id="50873-133">Ouvrez **WelcomeViewController.m** et ajoutez l’instruction `import` suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="50873-133">Open **WelcomeViewController.m** and add the following `import` statement to the top of the file.</span></span>

    ```objc
    #import "AuthenticationManager.h"
    ```

1. <span data-ttu-id="50873-134">Remplacez la fonction `signOut` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="50873-134">Replace the existing `signOut` function with the following.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. <span data-ttu-id="50873-135">Enregistrez vos modifications et redémarrez l’application dans simulateur.</span><span class="sxs-lookup"><span data-stu-id="50873-135">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="50873-136">Si vous vous connectez à l’application, un jeton d’accès doit s’afficher dans la fenêtre de sortie dans Xcode.</span><span class="sxs-lookup"><span data-stu-id="50873-136">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Capture d’écran de la fenêtre de sortie dans Xcode affichant un jeton d’accès](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="50873-138">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="50873-138">Get user details</span></span>

<span data-ttu-id="50873-139">Dans cette section, vous allez créer une classe d’aide pour contenir tous les appels à Microsoft Graph et mettre à jour la classe pour utiliser cette nouvelle classe afin d’obtenir l’utilisateur `WelcomeViewController` connecté.</span><span class="sxs-lookup"><span data-stu-id="50873-139">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="50873-140">Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **GraphManager**.</span><span class="sxs-lookup"><span data-stu-id="50873-140">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="50873-141">Choisissez **NSObject dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="50873-141">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="50873-142">Ouvrez **GraphManager.h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-142">Open **GraphManager.h** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="50873-143">Ouvrez **GraphManager.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-143">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="50873-144">Ouvrez **WelcomeViewController.m** et ajoutez les `#import` instructions suivantes en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="50873-144">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="50873-145">Ajoutez la propriété suivante à la déclaration `WelcomeViewController` d’interface.</span><span class="sxs-lookup"><span data-stu-id="50873-145">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="50873-146">Remplacez `viewDidLoad` l’existant par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="50873-146">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

<span data-ttu-id="50873-147">Si vous enregistrez vos modifications et redémarrez l’application maintenant, après la mise à jour de l’interface utilisateur avec le nom d’affichage et l’adresse e-mail de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="50873-147">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
