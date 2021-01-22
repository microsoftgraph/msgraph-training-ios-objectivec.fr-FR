<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f78d7-101">Dans cette section, vous allez ajouter la possibilité de créer des événements sur le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="f78d7-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="f78d7-102">Ouvrez **GraphManager.h** et ajoutez le code suivant au-dessus de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="f78d7-102">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^CreateEventCompletionBlock)(MSGraphEvent* _Nullable event,
                                               NSError* _Nullable error);
    ```

1. <span data-ttu-id="f78d7-103">Ajoutez le code suivant à la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="f78d7-103">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) createEventWithSubject: (NSString*) subject
                           andStart: (NSDate*) start
                             andEnd: (NSDate*) end
                       andAttendees: (NSArray<NSString*>* _Nullable) attendees
                            andBody: (NSString* _Nullable) body
                 andCompletionBlock: (CreateEventCompletionBlock) completion;
    ```

1. <span data-ttu-id="f78d7-104">Ouvrez **GraphManager.m** et ajoutez la fonction suivante pour créer un événement sur le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="f78d7-104">Open **GraphManager.m** and add the following function to create a new event on the user's calendar.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="CreateEventSnippet":::

1. <span data-ttu-id="f78d7-105">Créez un **fichier de classe Cocoa Touch** dans le dossier **GraphTutorial** nommé `NewEventViewController` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-105">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `NewEventViewController`.</span></span> <span data-ttu-id="f78d7-106">Choisissez **UIViewController dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="f78d7-106">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="f78d7-107">Ouvrez **NewEventViewController.h** et ajoutez le code suivant à l’intérieur de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="f78d7-107">Open **NewEventViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objectivec
    @property (nonatomic) IBOutlet UITextField* subject;
    @property (nonatomic) IBOutlet UITextField* attendees;
    @property (nonatomic) IBOutlet UIDatePicker* start;
    @property (nonatomic) IBOutlet UIDatePicker* end;
    @property (nonatomic) IBOutlet UITextView* body;
    ```

1. <span data-ttu-id="f78d7-108">Ouvrez **NewEventController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="f78d7-108">Open **NewEventController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/NewEventViewController.m" id="NewEventViewControllerSnippet":::

1. <span data-ttu-id="f78d7-109">Ouvrez **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="f78d7-109">Open **Main.storyboard**.</span></span> <span data-ttu-id="f78d7-110">Utilisez la **bibliothèque pour** faire glisser un contrôleur **de vue** vers le storyboard.</span><span class="sxs-lookup"><span data-stu-id="f78d7-110">Use the **Library** to drag a **View Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="f78d7-111">À **l’aide de la bibliothèque,** ajoutez une **barre de navigation** au contrôleur d’affichage.</span><span class="sxs-lookup"><span data-stu-id="f78d7-111">Using the **Library**, add a **Navigation Bar** to the view controller.</span></span>
1. <span data-ttu-id="f78d7-112">Double-cliquez sur **le titre dans** la barre de navigation et mettez-le à `New Event` jour.</span><span class="sxs-lookup"><span data-stu-id="f78d7-112">Double-click the **Title** in the navigation bar and update it to `New Event`.</span></span>
1. <span data-ttu-id="f78d7-113">À **l’aide de la bibliothèque,** ajoutez un élément de bouton **de** barre sur le côté gauche de la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="f78d7-113">Using the **Library**, add a **Bar Button Item** to the left-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="f78d7-114">Sélectionnez le nouveau bouton de barre, puis l’inspecteur **d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="f78d7-114">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="f78d7-115">Change **title** to `Cancel` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-115">Change **Title** to `Cancel`.</span></span>
1. <span data-ttu-id="f78d7-116">À **l’aide de la bibliothèque,** ajoutez un élément de bouton **de** barre à droite de la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="f78d7-116">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="f78d7-117">Sélectionnez le nouveau bouton de barre, puis l’inspecteur **d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="f78d7-117">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="f78d7-118">Change **title** to `Create` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-118">Change **Title** to `Create`.</span></span>
1. <span data-ttu-id="f78d7-119">Sélectionnez le contrôleur d’affichage, puis **l’inspecteur d’identité.**</span><span class="sxs-lookup"><span data-stu-id="f78d7-119">Select the view controller, then select the **Identity Inspector**.</span></span> <span data-ttu-id="f78d7-120">Change **class** to **NewEventViewController**.</span><span class="sxs-lookup"><span data-stu-id="f78d7-120">Change **Class** to **NewEventViewController**.</span></span>
1. <span data-ttu-id="f78d7-121">Ajoutez les contrôles suivants de **la bibliothèque** à l’affichage.</span><span class="sxs-lookup"><span data-stu-id="f78d7-121">Add the following controls from the **Library** to the view.</span></span>

    - <span data-ttu-id="f78d7-122">Ajoutez une **étiquette sous** la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="f78d7-122">Add a **Label** under the navigation bar.</span></span> <span data-ttu-id="f78d7-123">Définissez son texte sur `Subject` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-123">Set its text to `Subject`.</span></span>
    - <span data-ttu-id="f78d7-124">Ajoutez **un champ de texte** sous l’étiquette.</span><span class="sxs-lookup"><span data-stu-id="f78d7-124">Add a **Text Field** under the label.</span></span> <span data-ttu-id="f78d7-125">Définissez son **attribut d’espace** réservé sur `Subject` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-125">Set its **Placeholder** attribute to `Subject`.</span></span>
    - <span data-ttu-id="f78d7-126">Ajoutez une **étiquette** sous le champ de texte.</span><span class="sxs-lookup"><span data-stu-id="f78d7-126">Add a **Label** under the text field.</span></span> <span data-ttu-id="f78d7-127">Définissez son texte sur `Attendees` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-127">Set its text to `Attendees`.</span></span>
    - <span data-ttu-id="f78d7-128">Ajoutez **un champ de texte** sous l’étiquette.</span><span class="sxs-lookup"><span data-stu-id="f78d7-128">Add a **Text Field** under the label.</span></span> <span data-ttu-id="f78d7-129">Définissez son **attribut d’espace** réservé sur `Separate multiple entries with ;` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-129">Set its **Placeholder** attribute to `Separate multiple entries with ;`.</span></span>
    - <span data-ttu-id="f78d7-130">Ajoutez une **étiquette** sous le champ de texte.</span><span class="sxs-lookup"><span data-stu-id="f78d7-130">Add a **Label** under the text field.</span></span> <span data-ttu-id="f78d7-131">Définissez son texte sur `Start` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-131">Set its text to `Start`.</span></span>
    - <span data-ttu-id="f78d7-132">Ajoutez un **s sélectionneur de dates** sous l’étiquette.</span><span class="sxs-lookup"><span data-stu-id="f78d7-132">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="f78d7-133">Définissez **son style préféré sur** **Compact,** son **intervalle** **à 15 minutes** et sa hauteur sur **35**.</span><span class="sxs-lookup"><span data-stu-id="f78d7-133">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="f78d7-134">Ajoutez une **étiquette sous** le s sélectionneur de dates.</span><span class="sxs-lookup"><span data-stu-id="f78d7-134">Add a **Label** under the date picker.</span></span> <span data-ttu-id="f78d7-135">Définissez son texte sur `End` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-135">Set its text to `End`.</span></span>
    - <span data-ttu-id="f78d7-136">Ajoutez un **s sélectionneur de dates** sous l’étiquette.</span><span class="sxs-lookup"><span data-stu-id="f78d7-136">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="f78d7-137">Définissez **son style préféré sur** **Compact,** son **intervalle** **à 15 minutes** et sa hauteur sur **35**.</span><span class="sxs-lookup"><span data-stu-id="f78d7-137">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="f78d7-138">Ajoutez un **affichage texte sous** le s sélectionneur de dates.</span><span class="sxs-lookup"><span data-stu-id="f78d7-138">Add a **Text View** under the date picker.</span></span>

1. <span data-ttu-id="f78d7-139">Sélectionnez **le nouveau contrôleur d’affichage d’événements** et utilisez **l’Inspecteur de** connexion pour établir les connexions suivantes.</span><span class="sxs-lookup"><span data-stu-id="f78d7-139">Select the **New Event View Controller** and use the **Connection Inspector** to make the following connections.</span></span>

    - <span data-ttu-id="f78d7-140">Connectez **l’action d’annulation** reçue au **bouton annuler** la barre.</span><span class="sxs-lookup"><span data-stu-id="f78d7-140">Connect the **cancel** received action to the **Cancel** bar button.</span></span>
    - <span data-ttu-id="f78d7-141">Connectez **l’action reçue createEvent** au **bouton Créer une** barre.</span><span class="sxs-lookup"><span data-stu-id="f78d7-141">Connect the **createEvent** received action to the **Create** bar button.</span></span>
    - <span data-ttu-id="f78d7-142">Connectez la **sortie de** l’objet au premier champ de texte.</span><span class="sxs-lookup"><span data-stu-id="f78d7-142">Connect the **subject** outlet to the first text field.</span></span>
    - <span data-ttu-id="f78d7-143">Connectez **la sortie des participants** au deuxième champ de texte.</span><span class="sxs-lookup"><span data-stu-id="f78d7-143">Connect the **attendees** outlet to the second text field.</span></span>
    - <span data-ttu-id="f78d7-144">Connectez **la sortie de** début au premier s sélectionneur de dates.</span><span class="sxs-lookup"><span data-stu-id="f78d7-144">Connect the **start** outlet to the first date picker.</span></span>
    - <span data-ttu-id="f78d7-145">Connectez **la sortie** de fin au deuxième s picker de date.</span><span class="sxs-lookup"><span data-stu-id="f78d7-145">Connect the **end** outlet to the second date picker.</span></span>
    - <span data-ttu-id="f78d7-146">Connectez la **sortie du** corps à l’affichage texte.</span><span class="sxs-lookup"><span data-stu-id="f78d7-146">Connect the **body** outlet to the text view.</span></span>

1. <span data-ttu-id="f78d7-147">Ajoutez les contraintes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f78d7-147">Add the following constraints.</span></span>

    - <span data-ttu-id="f78d7-148">**Barre de navigation**</span><span class="sxs-lookup"><span data-stu-id="f78d7-148">**Navigation Bar**</span></span>
        - <span data-ttu-id="f78d7-149">Espace de pointe vers la zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-149">Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="f78d7-150">Espace à la fin de la zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-150">Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="f78d7-151">Espace supérieur vers la zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-151">Top space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="f78d7-152">Hauteur, valeur : 44</span><span class="sxs-lookup"><span data-stu-id="f78d7-152">Height, value: 44</span></span>
    - <span data-ttu-id="f78d7-153">**Étiquette d’objet**</span><span class="sxs-lookup"><span data-stu-id="f78d7-153">**Subject Label**</span></span>
        - <span data-ttu-id="f78d7-154">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-154">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-155">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-155">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-156">Espace supérieur de la barre de navigation, valeur : 20</span><span class="sxs-lookup"><span data-stu-id="f78d7-156">Top space to Navigation Bar, value: 20</span></span>
    - <span data-ttu-id="f78d7-157">**Champ de texte de l’objet**</span><span class="sxs-lookup"><span data-stu-id="f78d7-157">**Subject Text Field**</span></span>
        - <span data-ttu-id="f78d7-158">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-158">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-159">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-159">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-160">Espace supérieur à l’étiquette de l’objet, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-160">Top space to Subject Label, value: Standard</span></span>
    - <span data-ttu-id="f78d7-161">**Étiquette Attendees**</span><span class="sxs-lookup"><span data-stu-id="f78d7-161">**Attendees Label**</span></span>
        - <span data-ttu-id="f78d7-162">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-162">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-163">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-163">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-164">Espace supérieur au champ texte de l’objet, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-164">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="f78d7-165">**Champ de texte Attendees**</span><span class="sxs-lookup"><span data-stu-id="f78d7-165">**Attendees Text Field**</span></span>
        - <span data-ttu-id="f78d7-166">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-166">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-167">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-167">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-168">Espace supérieur à l’étiquette Attendees, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-168">Top space to Attendees Label, value: Standard</span></span>
    - <span data-ttu-id="f78d7-169">**Start Label**</span><span class="sxs-lookup"><span data-stu-id="f78d7-169">**Start Label**</span></span>
        - <span data-ttu-id="f78d7-170">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-170">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-171">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-171">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-172">Espace supérieur au champ texte de l’objet, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-172">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="f78d7-173">**Socheur de date de début**</span><span class="sxs-lookup"><span data-stu-id="f78d7-173">**Start Date Picker**</span></span>
        - <span data-ttu-id="f78d7-174">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-174">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-175">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-175">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-176">Espace supérieur à l’étiquette Attendees, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-176">Top space to Attendees Label, value: Standard</span></span>
        - <span data-ttu-id="f78d7-177">Hauteur, valeur : 35</span><span class="sxs-lookup"><span data-stu-id="f78d7-177">Height, value: 35</span></span>
    - <span data-ttu-id="f78d7-178">**End Label**</span><span class="sxs-lookup"><span data-stu-id="f78d7-178">**End Label**</span></span>
        - <span data-ttu-id="f78d7-179">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-179">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-180">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-180">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-181">Espace supérieur au s picker de date de début, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-181">Top space to Start Date Picker, value: Standard</span></span>
    - <span data-ttu-id="f78d7-182">**Socheur de date de fin**</span><span class="sxs-lookup"><span data-stu-id="f78d7-182">**End Date Picker**</span></span>
        - <span data-ttu-id="f78d7-183">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-183">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-184">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-184">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-185">Espace supérieur à l’étiquette de fin, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-185">Top space to End Label, value: Standard</span></span>
        - <span data-ttu-id="f78d7-186">Hauteur : 35</span><span class="sxs-lookup"><span data-stu-id="f78d7-186">Height: 35</span></span>
    - <span data-ttu-id="f78d7-187">**Affichage corps de texte**</span><span class="sxs-lookup"><span data-stu-id="f78d7-187">**Body Text View**</span></span>
        - <span data-ttu-id="f78d7-188">Espace de tête vers la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-188">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-189">Espace à la fin de la marge d’affichage, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-189">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="f78d7-190">Espace supérieur au s picker de date de fin, valeur : Standard</span><span class="sxs-lookup"><span data-stu-id="f78d7-190">Top space to End Date Picker, value: Standard</span></span>
        - <span data-ttu-id="f78d7-191">Espace inférieur pour afficher la marge, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="f78d7-191">Bottom space to View margin, value: 0</span></span>

    ![Capture d’écran du nouveau formulaire d’événement dans la storyboard](images/new-event-form.png)

1. <span data-ttu-id="f78d7-193">Sélectionnez **la scène de** calendrier, puis **l’inspecteur de connexions.**</span><span class="sxs-lookup"><span data-stu-id="f78d7-193">Select the **Calendar Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="f78d7-194">Sous **Segues déclenchés,** faites glisser le cercle non rempli en regard du **manuel** sur le nouveau contrôleur d’affichage d’événements dans le storyboard. </span><span class="sxs-lookup"><span data-stu-id="f78d7-194">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **New Event View Controller** on the storyboard.</span></span> <span data-ttu-id="f78d7-195">Sélectionnez **Présenter en mode mod** dans le menu déroulant.</span><span class="sxs-lookup"><span data-stu-id="f78d7-195">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="f78d7-196">Sélectionnez le segue que vous avez ajouté, puis sélectionnez **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="f78d7-196">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="f78d7-197">Définissez le **champ identificateur** sur `showEventForm` .</span><span class="sxs-lookup"><span data-stu-id="f78d7-197">Set the **Identifier** field to `showEventForm`.</span></span>
1. <span data-ttu-id="f78d7-198">Connectez **l’action reçue showNewEventForm** au bouton **+** de barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="f78d7-198">Connect the **showNewEventForm** received action to the **+** navigation bar button.</span></span>
1. <span data-ttu-id="f78d7-199">Enregistrez vos modifications, puis redémarrez l’application.</span><span class="sxs-lookup"><span data-stu-id="f78d7-199">Save your changes and restart the app.</span></span> <span data-ttu-id="f78d7-200">Go to the calendar page and tap the **+** button.</span><span class="sxs-lookup"><span data-stu-id="f78d7-200">Go to the calendar page and tap the **+** button.</span></span> <span data-ttu-id="f78d7-201">Remplissez le formulaire et **appuyez sur Créer** pour créer un événement.</span><span class="sxs-lookup"><span data-stu-id="f78d7-201">Fill in the form and tap **Create** to create a new event.</span></span>

    ![Capture d’écran du nouveau formulaire d’événement](images/create-event.png)
