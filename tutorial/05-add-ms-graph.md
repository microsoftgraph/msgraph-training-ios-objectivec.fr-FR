<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="11c49-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="11c49-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="11c49-102">Pour cette application, vous allez utiliser le [Kit de développement logiciel (SDK) Microsoft Graph pour effectuer des](https://github.com/microsoftgraph/msgraph-sdk-objc) appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="11c49-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="11c49-103">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="11c49-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="11c49-104">Dans cette section, vous allez étendre `GraphManager` la classe pour ajouter une fonction afin d’obtenir les événements de l' `CalendarViewController` utilisateur et la mise à jour pour utiliser ces nouvelles fonctions.</span><span class="sxs-lookup"><span data-stu-id="11c49-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="11c49-105">Ouvrez **GraphManager. h** et ajoutez le code suivant au- `@interface` dessus de la déclaration.</span><span class="sxs-lookup"><span data-stu-id="11c49-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. <span data-ttu-id="11c49-106">Ajoutez le code suivant à la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="11c49-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. <span data-ttu-id="11c49-107">Ouvrez **GraphManager. m** et ajoutez la fonction suivante à la `GraphManager` classe.</span><span class="sxs-lookup"><span data-stu-id="11c49-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock:(GetEventsCompletionBlock)completionBlock {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/events?%@&%@",
         MSGraphBaseURL,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by when they were created, newest first
         @"$orderby=createdDateTime+DESC"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // TEMPORARY
             completionBlock(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="11c49-108">Examinez le contenu du `getEventsWithCompletionBlock` code.</span><span class="sxs-lookup"><span data-stu-id="11c49-108">Consider what the code in `getEventsWithCompletionBlock` is doing.</span></span>
    >
    > - <span data-ttu-id="11c49-109">L’URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="11c49-109">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="11c49-110">Le `select` paramètre de requête limite les champs renvoyés pour chaque événement à seulement ceux que l’application utilisera réellement.</span><span class="sxs-lookup"><span data-stu-id="11c49-110">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="11c49-111">Le `orderby` paramètre de requête trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="11c49-111">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="11c49-112">Ouvrez **CalendarViewController. m** et remplacez l’intégralité de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="11c49-112">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface CalendarViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [GraphManager.instance
         getEventsWithCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error) {
                     // Show the error
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error getting events"
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

                 // TEMPORARY
                 self.calendarJSON.text = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
                 [self.calendarJSON sizeToFit];
             });
         }];
    }

    @end
    ```

1. <span data-ttu-id="11c49-113">Exécutez l’application, connectez-vous, puis appuyez sur l’élément de navigation **calendrier** dans le menu.</span><span class="sxs-lookup"><span data-stu-id="11c49-113">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="11c49-114">Vous devriez voir un dump JSON des événements dans l’application.</span><span class="sxs-lookup"><span data-stu-id="11c49-114">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="11c49-115">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="11c49-115">Display the results</span></span>

<span data-ttu-id="11c49-116">À présent, vous pouvez remplacer le vidage JSON par un texte pour afficher les résultats de manière conviviale.</span><span class="sxs-lookup"><span data-stu-id="11c49-116">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="11c49-117">Dans cette section, vous allez modifier la `getEventsWithCompletionBlock` fonction pour renvoyer des objets fortement typés et modifier `CalendarViewController` pour qu’elle affiche les événements à l’aide d’un affichage tableau.</span><span class="sxs-lookup"><span data-stu-id="11c49-117">In this section, you will modify the `getEventsWithCompletionBlock` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="11c49-118">Mettre à jour getEvents</span><span class="sxs-lookup"><span data-stu-id="11c49-118">Update getEvents</span></span>

1. <span data-ttu-id="11c49-119">Ouvrez **GraphManager. h**.</span><span class="sxs-lookup"><span data-stu-id="11c49-119">Open **GraphManager.h**.</span></span> <span data-ttu-id="11c49-120">Modifiez la `GetEventsCompletionBlock` définition de type comme suit.</span><span class="sxs-lookup"><span data-stu-id="11c49-120">Change the `GetEventsCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="11c49-121">Ouvrez **GraphManager. m**.</span><span class="sxs-lookup"><span data-stu-id="11c49-121">Open **GraphManager.m**.</span></span> <span data-ttu-id="11c49-122">Remplacez la `completionBlock(data, nil);` ligne dans la `getEventsWithCompletionBlock` fonction par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="11c49-122">Replace the `completionBlock(data, nil);` line in the `getEventsWithCompletionBlock` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetEventsSnippet" highlight="24-43":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="11c49-123">Mettre à jour CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="11c49-123">Update CalendarViewController</span></span>

1. <span data-ttu-id="11c49-124">Créez un fichier de **classe Touch de cacao** dans le projet **GraphTutorial** nommé `CalendarTableViewCell`.</span><span class="sxs-lookup"><span data-stu-id="11c49-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="11c49-125">Choisissez **UITableViewCell** dans la sous- **classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="11c49-125">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="11c49-126">Ouvrez **CalendarTableViewCell. h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="11c49-126">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="11c49-127">Ouvrez **CalendarTableViewCell. m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="11c49-127">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="11c49-128">Ouvrez **main. Storyboard** et localisez la **scène de calendrier**.</span><span class="sxs-lookup"><span data-stu-id="11c49-128">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="11c49-129">Sélectionnez l' **affichage** dans la **scène de calendrier** et supprimez-le.</span><span class="sxs-lookup"><span data-stu-id="11c49-129">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Capture d’écran de l’affichage dans la scène de calendrier](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="11c49-131">Ajoutez un **affichage tableau** de la **bibliothèque** à la **scène calendrier**.</span><span class="sxs-lookup"><span data-stu-id="11c49-131">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="11c49-132">Sélectionnez l’affichage tableau, puis sélectionnez l' **inspecteur d’attributs**.</span><span class="sxs-lookup"><span data-stu-id="11c49-132">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="11c49-133">Définissez les **cellules prototype** sur **1**.</span><span class="sxs-lookup"><span data-stu-id="11c49-133">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="11c49-134">Utilisez la **bibliothèque** pour ajouter trois **étiquettes** à la cellule prototype.</span><span class="sxs-lookup"><span data-stu-id="11c49-134">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="11c49-135">Sélectionnez la cellule prototype, puis l' **inspecteur d’identité**.</span><span class="sxs-lookup"><span data-stu-id="11c49-135">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="11c49-136">Modifiez **Class** en **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="11c49-136">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="11c49-137">Sélectionnez l' **inspecteur d’attributs** et définissez **identificateur** sur `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="11c49-137">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="11c49-138">Une fois le **EventCell** sélectionné, sélectionnez l' **inspecteur** de connexions `durationLabel`et `organizerLabel`Connectez- `subjectLabel` vous, puis les étiquettes que vous avez ajoutées à la cellule sur la table de montage séquentiel.</span><span class="sxs-lookup"><span data-stu-id="11c49-138">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="11c49-139">Définissez les propriétés et les contraintes sur les trois étiquettes comme suit.</span><span class="sxs-lookup"><span data-stu-id="11c49-139">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="11c49-140">**Étiquette de l’objet**</span><span class="sxs-lookup"><span data-stu-id="11c49-140">**Subject Label**</span></span>
        - <span data-ttu-id="11c49-141">Ajouter une contrainte : espace de début à la marge de début de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-141">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-142">Ajouter une contrainte : espace de fin à la marge de fin de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-142">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-143">Ajouter une contrainte : espace à la marge supérieure de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-143">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="11c49-144">**Étiquette de l’organisateur**</span><span class="sxs-lookup"><span data-stu-id="11c49-144">**Organizer Label**</span></span>
        - <span data-ttu-id="11c49-145">Police : système 12,0</span><span class="sxs-lookup"><span data-stu-id="11c49-145">Font: System 12.0</span></span>
        - <span data-ttu-id="11c49-146">Ajouter une contrainte : espace de début à la marge de début de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-146">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-147">Ajouter une contrainte : espace de fin à la marge de fin de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-147">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-148">Ajouter une contrainte : espace vers le bas de l’étiquette d’objet, valeur : standard</span><span class="sxs-lookup"><span data-stu-id="11c49-148">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="11c49-149">**Étiquette de durée**</span><span class="sxs-lookup"><span data-stu-id="11c49-149">**Duration Label**</span></span>
        - <span data-ttu-id="11c49-150">Police : système 12,0</span><span class="sxs-lookup"><span data-stu-id="11c49-150">Font: System 12.0</span></span>
        - <span data-ttu-id="11c49-151">Couleur : gris foncé</span><span class="sxs-lookup"><span data-stu-id="11c49-151">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="11c49-152">Ajouter une contrainte : espace de début à la marge de début de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-152">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-153">Ajouter une contrainte : espace de fin à la marge de fin de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="11c49-153">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="11c49-154">Ajouter une contrainte : espace vers le bas de l’étiquette de l’organisateur, valeur : standard</span><span class="sxs-lookup"><span data-stu-id="11c49-154">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="11c49-155">Ajouter une contrainte : espace inférieur au contenu afficher la marge inférieure, valeur : 8</span><span class="sxs-lookup"><span data-stu-id="11c49-155">Add constraint: Bottom space to Content View Bottom Margin, value: 8</span></span>

    ![Capture d’écran de la mise en page de cellule prototype](./images/prototype-cell-layout.png)

1. <span data-ttu-id="11c49-157">Ouvrez **CalendarViewController. h** et supprimez `calendarJSON` la propriété.</span><span class="sxs-lookup"><span data-stu-id="11c49-157">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>
1. <span data-ttu-id="11c49-158">Modifiez la `@interface` déclaration comme suit.</span><span class="sxs-lookup"><span data-stu-id="11c49-158">Change the `@interface` declaration to the following.</span></span>

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. <span data-ttu-id="11c49-159">Ouvrez **CalendarViewController. m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="11c49-159">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewSnippet":::

1. <span data-ttu-id="11c49-160">Exécutez l’application, connectez-vous, puis appuyez sur l’onglet **calendrier** . La liste des événements doit s’afficher.</span><span class="sxs-lookup"><span data-stu-id="11c49-160">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/calendar-list.png)
