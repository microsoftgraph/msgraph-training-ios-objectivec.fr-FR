<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b4d18-101">Commencez par créer un projet Objective-C.</span><span class="sxs-lookup"><span data-stu-id="b4d18-101">Begin by creating a new Objective-C project.</span></span>

1. <span data-ttu-id="b4d18-102">Ouvrez Xcode.</span><span class="sxs-lookup"><span data-stu-id="b4d18-102">Open Xcode.</span></span> <span data-ttu-id="b4d18-103">Dans **le** menu Fichier, **sélectionnez Nouveau,** puis **Projet**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="b4d18-104">Choisissez le modèle **d’application** et sélectionnez **Suivant.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-104">Choose the **App** template and select **Next**.</span></span>

    ![Capture d’écran de la boîte de dialogue de sélection du modèle Xcode](./images/xcode-select-template.png)

1. <span data-ttu-id="b4d18-106">Définissez **le nom du produit** et la `GraphTutorial` **langue** sur **Objective-C**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Objective-C**.</span></span>
1. <span data-ttu-id="b4d18-107">Remplissez les champs restants et sélectionnez **Suivant**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="b4d18-108">Choisissez un emplacement pour le projet, puis sélectionnez **Créer.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="b4d18-109">Installer les dépendances</span><span class="sxs-lookup"><span data-stu-id="b4d18-109">Install dependencies</span></span>

<span data-ttu-id="b4d18-110">Avant de passer à autre chose, installez des dépendances supplémentaires que vous utiliserez ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="b4d18-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="b4d18-111">[Bibliothèque d’authentification Microsoft (MSAL) pour iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) pour l’authentification auprès d’Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b4d18-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="b4d18-112">[SDK Microsoft Graph pour l’objectif C](https://github.com/microsoftgraph/msgraph-sdk-objc) pour effectuer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b4d18-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="b4d18-113">[Microsoft Graph Models SDK pour l’objectif C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) pour les objets fortement typés représentant des ressources Microsoft Graph telles que des utilisateurs ou des événements.</span><span class="sxs-lookup"><span data-stu-id="b4d18-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="b4d18-114">Quittez Xcode.</span><span class="sxs-lookup"><span data-stu-id="b4d18-114">Quit Xcode.</span></span>
1. <span data-ttu-id="b4d18-115">Ouvrez Terminal et modifiez le répertoire à l’emplacement de **votre projet GraphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="b4d18-116">Exécutez la commande suivante pour créer un podfile.</span><span class="sxs-lookup"><span data-stu-id="b4d18-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="b4d18-117">Ouvrez le podfile et ajoutez les lignes suivantes juste après la `use_frameworks!` ligne.</span><span class="sxs-lookup"><span data-stu-id="b4d18-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="b4d18-118">Enregistrez le podfile, puis exécutez la commande suivante pour installer les dépendances.</span><span class="sxs-lookup"><span data-stu-id="b4d18-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="b4d18-119">Une fois la commande terminée, ouvrez l’espace **GraphTutorial.xcworkspace** nouvellement créé dans Xcode.</span><span class="sxs-lookup"><span data-stu-id="b4d18-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="b4d18-120">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="b4d18-120">Design the app</span></span>

<span data-ttu-id="b4d18-121">Dans cette section, vous allez créer les vues de l’application : une page de signature, un navigateur de barre d’onglets, une page d’accueil et une page de calendrier.</span><span class="sxs-lookup"><span data-stu-id="b4d18-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="b4d18-122">Vous allez également créer une superposition d’indicateur d’activité.</span><span class="sxs-lookup"><span data-stu-id="b4d18-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="b4d18-123">Page Créer une signature</span><span class="sxs-lookup"><span data-stu-id="b4d18-123">Create sign in page</span></span>

1. <span data-ttu-id="b4d18-124">Développez **le dossier GraphTutorial** dans Xcode, puis sélectionnez le **fichier ViewController.m.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-124">Expand the **GraphTutorial** folder in Xcode, then select the **ViewController.m** file.</span></span>
1. <span data-ttu-id="b4d18-125">Dans **l’Inspecteur de** fichier, modifiez **le nom** du fichier en `SignInViewController.m` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.m`.</span></span>

    ![Capture d’écran de l’Inspecteur de fichier](./images/rename-file.png)

1. <span data-ttu-id="b4d18-127">Ouvrez **SignInViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-127">Open **SignInViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"

    @interface SignInViewController ()

    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.
    }

    - (IBAction)signIn {
        [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
    }
    @end
    ```

1. <span data-ttu-id="b4d18-128">Sélectionnez **le fichier ViewController.h.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-128">Select the **ViewController.h** file.</span></span>
1. <span data-ttu-id="b4d18-129">Dans **l’Inspecteur de** fichier, modifiez **le nom** du fichier en `SignInViewController.h` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-129">In the **File Inspector**, change the **Name** of the file to `SignInViewController.h`.</span></span>
1. <span data-ttu-id="b4d18-130">Ouvrez **SignInViewController.h** et modifiez toutes les instances `ViewController` de `SignInViewController` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-130">Open **SignInViewController.h** and change all instances of `ViewController` to `SignInViewController`.</span></span>

1. <span data-ttu-id="b4d18-131">Ouvrez **le fichier Main.storyboard.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-131">Open the **Main.storyboard** file.</span></span>
1. <span data-ttu-id="b4d18-132">Développez **La scène du contrôleur de** vue, puis sélectionnez Contrôleur de **vue.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-132">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![Capture d’écran de Xcode avec le contrôleur d’affichage sélectionné](./images/storyboard-select-view-controller.png)

1. <span data-ttu-id="b4d18-134">Sélectionnez **l’Inspecteur d’identité,** puis modifiez **la** classe de la catégorie à **SignInViewController**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-134">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![Capture d’écran de l’Inspecteur d’identité](./images/change-class.png)

1. <span data-ttu-id="b4d18-136">Sélectionnez **la bibliothèque,** puis faites glisser un **bouton sur** le contrôleur **d’affichage de la signature.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-136">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Capture d’écran de la bibliothèque dans Xcode](./images/add-button-to-view.png)

1. <span data-ttu-id="b4d18-138">Une fois le bouton sélectionné, sélectionnez  l’Inspecteur d’attributs et modifiez le titre du bouton sur  `Sign In` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-138">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Capture d’écran du champ Titre dans l’inspecteur d’attributs dans Xcode](./images/set-button-title.png)

1. <span data-ttu-id="b4d18-140">Une fois le bouton sélectionné, sélectionnez le bouton **Aligner** en bas du storyboard.</span><span class="sxs-lookup"><span data-stu-id="b4d18-140">With the button selected, select the **Align** button at the bottom of the storyboard.</span></span> <span data-ttu-id="b4d18-141">Sélectionnez les contraintes **Horizontalement** dans le conteneur et **Verticalement** dans les contraintes de conteneur, laissez leurs valeurs sous la forme 0, puis sélectionnez Ajouter **2 contraintes.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-141">Select both the **Horizontally in container** and **Vertically in container** constraints, leave their values as 0, then select **Add 2 constraints**.</span></span>

    ![Capture d’écran des paramètres de contraintes d’alignement dans Xcode](./images/add-alignment-constraints.png)

1. <span data-ttu-id="b4d18-143">Sélectionnez **le contrôleur d’affichage de** connexion, puis **l’inspecteur de connexions.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-143">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="b4d18-144">Sous **Actions reçues,** faites glisser le cercle non rempli en dessous de **la** signature sur le bouton.</span><span class="sxs-lookup"><span data-stu-id="b4d18-144">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="b4d18-145">Sélectionnez **Toucher à l’intérieur** dans le menu déroulant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-145">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Capture d’écran du glissement de l’action signIn vers le bouton dans Xcode](./images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a><span data-ttu-id="b4d18-147">Créer une barre d’onglets</span><span class="sxs-lookup"><span data-stu-id="b4d18-147">Create tab bar</span></span>

1. <span data-ttu-id="b4d18-148">Sélectionnez la **bibliothèque,** puis faites glisser un contrôleur **de barre** d’onglets vers le storyboard.</span><span class="sxs-lookup"><span data-stu-id="b4d18-148">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="b4d18-149">Sélectionnez **le contrôleur d’affichage de** connexion, puis **l’inspecteur de connexions.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-149">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="b4d18-150">Sous **Segues déclenchés,** faites glisser le cercle non rempli à côté du **manuel** sur le contrôleur de barre de **tabulation** dans le storyboard.</span><span class="sxs-lookup"><span data-stu-id="b4d18-150">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="b4d18-151">Sélectionnez **Présenter en mode mod** dans le menu déroulant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-151">Select **Present Modally** in the pop-up menu.</span></span>

    ![Capture d’écran du glissement d’un segue manuel vers le nouveau contrôleur de barre d’onglets dans Xcode](./images/add-segue.png)

1. <span data-ttu-id="b4d18-153">Sélectionnez le segue que vous avez ajouté, puis sélectionnez **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-153">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="b4d18-154">Définissez le **champ identificateur** sur , et `userSignedIn` **définissez Présentation** en plein **écran**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-154">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Capture d’écran du champ identificateur dans l’inspecteur d’attributs dans Xcode](./images/set-segue-identifier.png)

1. <span data-ttu-id="b4d18-156">Sélectionnez **la scène d’élément 1,** puis l’inspecteur **de connexions.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-156">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="b4d18-157">Sous **Segues déclenchés,** faites glisser le  cercle non rempli en regard du manuel sur le contrôleur d’affichage de la signature **dans** le storyboard.</span><span class="sxs-lookup"><span data-stu-id="b4d18-157">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="b4d18-158">Sélectionnez **Présenter en mode mod** dans le menu déroulant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-158">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="b4d18-159">Sélectionnez le segue que vous avez ajouté, puis sélectionnez **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-159">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="b4d18-160">Définissez le **champ identificateur** sur , et `userSignedOut` **définissez Présentation** en plein **écran**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-160">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="b4d18-161">Créer une page d’accueil</span><span class="sxs-lookup"><span data-stu-id="b4d18-161">Create welcome page</span></span>

1. <span data-ttu-id="b4d18-162">Sélectionnez **le fichier Assets.x vhs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-162">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="b4d18-163">Dans le menu **Éditeur,** **sélectionnez Ajouter un nouvel actif,** puis Jeu **d’images.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-163">On the **Editor** menu, select **Add New Asset**, then **Image Set**.</span></span>
1. <span data-ttu-id="b4d18-164">Sélectionnez la nouvelle **image et** utilisez l’inspecteur **d’attributs** pour définir son **nom** `DefaultUserPhoto` sur .</span><span class="sxs-lookup"><span data-stu-id="b4d18-164">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="b4d18-165">Ajoutez n’importe quelle image que vous souhaitez servir de photo de profil utilisateur par défaut.</span><span class="sxs-lookup"><span data-stu-id="b4d18-165">Add any image you like to serve as a default user profile photo.</span></span>

    ![Capture d’écran de l’affichage actif Jeu d’images dans Xcode](./images/add-default-user-photo.png)

1. <span data-ttu-id="b4d18-167">Créez un **fichier de classe Cocoa Touch** dans le dossier **GraphTutorial** nommé `WelcomeViewController` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-167">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="b4d18-168">Choisissez **UIViewController dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="b4d18-168">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="b4d18-169">Ouvrez **WelcomeViewController.h** et ajoutez le code suivant à l’intérieur de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="b4d18-169">Open **WelcomeViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UIImageView *userProfilePhoto;
    @property (nonatomic) IBOutlet UILabel *userDisplayName;
    @property (nonatomic) IBOutlet UILabel *userEmail;
    ```

1. <span data-ttu-id="b4d18-170">Ouvrez **WelcomeViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-170">Open **WelcomeViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"

    @interface WelcomeViewController ()

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];
        self.userDisplayName.text = @"Default User";
        [self.userDisplayName sizeToFit];
        self.userEmail.text = @"default@contoso.com";
        [self.userEmail sizeToFit];
    }

    - (IBAction)signOut {
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="b4d18-171">Ouvrez **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-171">Open **Main.storyboard**.</span></span> <span data-ttu-id="b4d18-172">Sélectionnez la **scène d’élément 1,** puis l’inspecteur **d’identité.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-172">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="b4d18-173">Modifiez **la valeur** de classe en **WelcomeViewController**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-173">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="b4d18-174">À **l’aide de la bibliothèque**, ajoutez les éléments suivants à la **scène d’élément 1.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-174">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="b4d18-175">Un **affichage d’image**</span><span class="sxs-lookup"><span data-stu-id="b4d18-175">One **Image View**</span></span>
    - <span data-ttu-id="b4d18-176">Deux **étiquettes**</span><span class="sxs-lookup"><span data-stu-id="b4d18-176">Two **Labels**</span></span>
    - <span data-ttu-id="b4d18-177">Un **bouton**</span><span class="sxs-lookup"><span data-stu-id="b4d18-177">One **Button**</span></span>

1. <span data-ttu-id="b4d18-178">À **l’aide de l’Inspecteur de connexions,** établir les connexions suivantes.</span><span class="sxs-lookup"><span data-stu-id="b4d18-178">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="b4d18-179">Lier la **sortie userDisplayName** à la première étiquette.</span><span class="sxs-lookup"><span data-stu-id="b4d18-179">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="b4d18-180">Lier la **sortie userEmail** à la deuxième étiquette.</span><span class="sxs-lookup"><span data-stu-id="b4d18-180">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="b4d18-181">Lier la **sortie userProfilePhoto** à la vue d’image.</span><span class="sxs-lookup"><span data-stu-id="b4d18-181">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="b4d18-182">Lier **l’action reçue de** connexion à l’action Touch Up Inside du **bouton.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-182">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="b4d18-183">Sélectionnez l’affichage d’image, puis **l’inspecteur de taille.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-183">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="b4d18-184">Définissez **la largeur** et la **hauteur** sur 196.</span><span class="sxs-lookup"><span data-stu-id="b4d18-184">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="b4d18-185">Utilisez le **bouton Aligner** pour ajouter la contrainte **Horizontalement dans le** conteneur avec la valeur 0.</span><span class="sxs-lookup"><span data-stu-id="b4d18-185">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="b4d18-186">Utilisez le **bouton Ajouter de nouvelles contraintes** (à côté du bouton **Aligner)** pour ajouter les contraintes suivantes :</span><span class="sxs-lookup"><span data-stu-id="b4d18-186">Use the **Add New Constraints** button (next to the **Align** button) to add the following constraints:</span></span>

    - <span data-ttu-id="b4d18-187">Aligner le haut sur : zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="b4d18-187">Align Top to: Safe Area, value: 0</span></span>
    - <span data-ttu-id="b4d18-188">Espace inférieur à : Nom d’affichage de l’utilisateur, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="b4d18-188">Bottom Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="b4d18-189">Hauteur, valeur : 196</span><span class="sxs-lookup"><span data-stu-id="b4d18-189">Height, value: 196</span></span>
    - <span data-ttu-id="b4d18-190">Largeur, valeur : 196</span><span class="sxs-lookup"><span data-stu-id="b4d18-190">Width, value: 196</span></span>

    ![Capture d’écran des nouveaux paramètres de contraintes dans Xcode](./images/add-new-constraints.png)

1. <span data-ttu-id="b4d18-192">Sélectionnez la première étiquette, puis utilisez le bouton **Aligner** pour ajouter la contrainte **Horizontalement** dans le conteneur avec la valeur 0.</span><span class="sxs-lookup"><span data-stu-id="b4d18-192">Select the first label, then use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="b4d18-193">Utilisez le **bouton Ajouter de nouvelles contraintes** pour ajouter les contraintes suivantes :</span><span class="sxs-lookup"><span data-stu-id="b4d18-193">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="b4d18-194">Espace supérieur vers : Photo de profil utilisateur, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="b4d18-194">Top Space to: User Profile Photo, value: Standard</span></span>
    - <span data-ttu-id="b4d18-195">Bottom Space to: User Email, value: Standard</span><span class="sxs-lookup"><span data-stu-id="b4d18-195">Bottom Space to: User Email, value: Standard</span></span>

1. <span data-ttu-id="b4d18-196">Sélectionnez la deuxième étiquette, puis **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-196">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="b4d18-197">Modifiez **la couleur** en gris **foncé** et modifiez la **police** sur **System 12.0**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-197">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="b4d18-198">Utilisez le **bouton Aligner** pour ajouter la contrainte **Horizontalement dans le** conteneur avec la valeur 0.</span><span class="sxs-lookup"><span data-stu-id="b4d18-198">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="b4d18-199">Utilisez le **bouton Ajouter de nouvelles contraintes** pour ajouter les contraintes suivantes :</span><span class="sxs-lookup"><span data-stu-id="b4d18-199">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="b4d18-200">Espace supérieur : Nom d’affichage de l’utilisateur, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="b4d18-200">Top Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="b4d18-201">Espace inférieur pour : Se sortir, valeur : 14</span><span class="sxs-lookup"><span data-stu-id="b4d18-201">Bottom Space to: Sign Out, value: 14</span></span>

1. <span data-ttu-id="b4d18-202">Sélectionnez le bouton, puis l’inspecteur **d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-202">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="b4d18-203">Changez **le titre** en `Sign Out` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-203">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="b4d18-204">Utilisez le **bouton Aligner** pour ajouter la contrainte **Horizontalement dans le** conteneur avec la valeur 0.</span><span class="sxs-lookup"><span data-stu-id="b4d18-204">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="b4d18-205">Utilisez le **bouton Ajouter de nouvelles contraintes** pour ajouter les contraintes suivantes :</span><span class="sxs-lookup"><span data-stu-id="b4d18-205">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="b4d18-206">Espace supérieur à : Courrier électronique de l’utilisateur, valeur : 14</span><span class="sxs-lookup"><span data-stu-id="b4d18-206">Top Space to: User Email, value: 14</span></span>

1. <span data-ttu-id="b4d18-207">Sélectionnez l’élément de barre d’onglets en bas de la scène, puis sélectionnez **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-207">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="b4d18-208">Changez **le titre** en `Me` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-208">Change the **Title** to `Me`.</span></span>
1. <span data-ttu-id="b4d18-209">Dans le menu **Éditeur,** sélectionnez Résoudre les problèmes de disposition **automatique,** puis sélectionnez Ajouter des **contraintes manquantes** sous Tous les affichages dans le contrôleur **d’affichage d’accueil.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-209">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="b4d18-210">La scène d’accueil doit ressembler à ceci une fois que vous avez terminé.</span><span class="sxs-lookup"><span data-stu-id="b4d18-210">The welcome scene should look similar to this once you're done.</span></span>

![Capture d’écran de la disposition de la scène d’accueil](./images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="b4d18-212">Créer une page de calendrier</span><span class="sxs-lookup"><span data-stu-id="b4d18-212">Create calendar page</span></span>

1. <span data-ttu-id="b4d18-213">Créez un **fichier de classe Cocoa Touch** dans le dossier **GraphTutorial** nommé `CalendarViewController` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-213">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="b4d18-214">Choisissez **UIViewController dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="b4d18-214">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="b4d18-215">Ouvrez **CalendarViewController.h** et ajoutez le code suivant à l’intérieur de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="b4d18-215">Open **CalendarViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UITextView *calendarJSON;
    ```

1. <span data-ttu-id="b4d18-216">Ouvrez **CalendarViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-216">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"

    @interface CalendarViewController ()

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.calendarJSON.text = @"Calendar";
        [self.calendarJSON sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="b4d18-217">Ouvrez **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-217">Open **Main.storyboard**.</span></span> <span data-ttu-id="b4d18-218">Sélectionnez **la scène d’élément 2,** puis l’inspecteur **d’identité.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-218">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="b4d18-219">Modifiez **la valeur de** classe en **CalendarViewController**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-219">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="b4d18-220">À **l’aide de la bibliothèque**, ajoutez **un affichage texte** à la **scène d’élément 2.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-220">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="b4d18-221">Sélectionnez l’affichage texte que vous avez ajouté.</span><span class="sxs-lookup"><span data-stu-id="b4d18-221">Select the text view you just added.</span></span> <span data-ttu-id="b4d18-222">On the **Editor**, choose **Embed In**, then **Scroll View**.</span><span class="sxs-lookup"><span data-stu-id="b4d18-222">On the **Editor**, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="b4d18-223">À **l’aide de l’Inspecteur de connexions,** connectez la sortie **calendarJSON** à l’affichage texte.</span><span class="sxs-lookup"><span data-stu-id="b4d18-223">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. <span data-ttu-id="b4d18-224">Sélectionnez l’élément de barre d’onglets en bas de la scène, puis sélectionnez **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-224">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="b4d18-225">Changez **le titre** en `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-225">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="b4d18-226">Dans le menu **Éditeur,** sélectionnez Résoudre les problèmes de disposition **automatique,** puis sélectionnez Ajouter des **contraintes manquantes** sous Tous les affichages dans le contrôleur **d’affichage d’accueil.**</span><span class="sxs-lookup"><span data-stu-id="b4d18-226">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="b4d18-227">La scène de calendrier doit ressembler à ceci une fois que vous avez terminé.</span><span class="sxs-lookup"><span data-stu-id="b4d18-227">The calendar scene should look similar to this once you're done.</span></span>

![Capture d’écran de la disposition de la scène calendrier](./images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="b4d18-229">Créer un indicateur d’activité</span><span class="sxs-lookup"><span data-stu-id="b4d18-229">Create activity indicator</span></span>

1. <span data-ttu-id="b4d18-230">Créez un **fichier de classe Cocoa Touch** dans le dossier **GraphTutorial** nommé `SpinnerViewController` .</span><span class="sxs-lookup"><span data-stu-id="b4d18-230">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="b4d18-231">Choisissez **UIViewController dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="b4d18-231">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="b4d18-232">Ouvrez **SpinnerViewController.h** et ajoutez le code suivant à l’intérieur de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="b4d18-232">Open **SpinnerViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    - (void) startWithContainer:(UIViewController*) container;
    - (void) stop;
    ```

1. <span data-ttu-id="b4d18-233">Ouvrez **SpinnerViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b4d18-233">Open **SpinnerViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.m" id="SpinnerViewSnippet":::

## <a name="test-the-app"></a><span data-ttu-id="b4d18-234">Tester l'application</span><span class="sxs-lookup"><span data-stu-id="b4d18-234">Test the app</span></span>

<span data-ttu-id="b4d18-235">Enregistrez vos modifications et lancez l’application.</span><span class="sxs-lookup"><span data-stu-id="b4d18-235">Save your changes and launch the app.</span></span> <span data-ttu-id="b4d18-236">Vous devriez être en mesure de vous déplacer entre les écrans à l’aide des boutons **Se** connectez et se connectez **et** de la barre d’onglets.</span><span class="sxs-lookup"><span data-stu-id="b4d18-236">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![Captures d’écran de l’application](./images/app-screens.png)
