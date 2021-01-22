<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="06e40-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="06e40-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="06e40-102">Pour cette application, vous allez utiliser le [SDK Microsoft Graph pour l’objectif C](https://github.com/microsoftgraph/msgraph-sdk-objc) pour effectuer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="06e40-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="06e40-103">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="06e40-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="06e40-104">Dans cette section, vous allez étendre la classe pour ajouter une fonction afin d’obtenir les événements de l’utilisateur pour la semaine en cours et mettre à jour pour `GraphManager` `CalendarViewController` utiliser cette nouvelle fonction.</span><span class="sxs-lookup"><span data-stu-id="06e40-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use this new function.</span></span>

1. <span data-ttu-id="06e40-105">Ouvrez **GraphManager.h** et ajoutez le code suivant au-dessus de la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="06e40-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. <span data-ttu-id="06e40-106">Ajoutez le code suivant à la `@interface` déclaration.</span><span class="sxs-lookup"><span data-stu-id="06e40-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. <span data-ttu-id="06e40-107">Ouvrez **GraphManager.m** et ajoutez la fonction suivante à la `GraphManager` classe.</span><span class="sxs-lookup"><span data-stu-id="06e40-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString *) viewStart
                              endingAt: (NSString *) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion {
        // Set calendar view start and end parameters
        NSString* viewStartEndString =
        [NSString stringWithFormat:@"startDateTime=%@&endDateTime=%@",
         viewStart,
         viewEnd];

        // GET /me/calendarview
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/calendarview?%@&%@&%@&%@",
         MSGraphBaseURL,
         viewStartEndString,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by start time
         @"$orderby=start/dateTime",
         // Request at most 25 results
         @"$top=25"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        NSString* preferHeader =
        [NSString stringWithFormat:@"outlook.timezone=\"%@\"",
         self.graphTimeZone];
        [eventsRequest addValue:preferHeader forHTTPHeaderField:@"Prefer"];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completion(nil, error);
                 return;
             }

             // TEMPORARY
             completion(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="06e40-108">Réfléchissez à ce que fait `getCalendarViewStartingAt` le code.</span><span class="sxs-lookup"><span data-stu-id="06e40-108">Consider what the code in `getCalendarViewStartingAt` is doing.</span></span>
    >
    > - <span data-ttu-id="06e40-109">L’URL qui sera appelée est `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="06e40-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="06e40-110">Les `startDateTime` `endDateTime` paramètres de requête définissent le début et la fin de l’affichage Calendrier.</span><span class="sxs-lookup"><span data-stu-id="06e40-110">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="06e40-111">Le paramètre de requête limite les champs renvoyés pour chaque événement à ceux que `select` l’affichage utilisera réellement.</span><span class="sxs-lookup"><span data-stu-id="06e40-111">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="06e40-112">Le `orderby` paramètre de requête trie les résultats par heure de début.</span><span class="sxs-lookup"><span data-stu-id="06e40-112">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="06e40-113">Le `top` paramètre de requête demande 25 résultats par page.</span><span class="sxs-lookup"><span data-stu-id="06e40-113">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="06e40-114">L’en-tête indique à Microsoft Graph de renvoyer les heures de début et de fin de chaque événement dans le `Prefer: outlook.timezone` fuseau horaire de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="06e40-114">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="06e40-115">Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **GraphToIana**.</span><span class="sxs-lookup"><span data-stu-id="06e40-115">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphToIana**.</span></span> <span data-ttu-id="06e40-116">Choisissez **NSObject dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="06e40-116">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06e40-117">Ouvrez **GraphToIana.h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-117">Open **GraphToIana.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. <span data-ttu-id="06e40-118">Ouvrez **GraphToIana.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-118">Open **GraphToIana.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    <span data-ttu-id="06e40-119">Cette opération permet de rechercher simplement un identificateur de fuseau horaire IANA basé sur le nom de fuseau horaire renvoyé par Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="06e40-119">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="06e40-120">Ouvrez **CalendarViewController.m** et remplacez tout son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-120">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import "GraphToIana.h"
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

        // Calculate the start and end of the current week
        NSString* timeZoneId = [GraphToIana
                                getIanaIdentifierFromGraphIdentifier:
                                [GraphManager.instance graphTimeZone]];

        NSDate* now = [NSDate date];
        NSCalendar* calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
        NSTimeZone* timeZone = [NSTimeZone timeZoneWithName:timeZoneId];
        [calendar setTimeZone:timeZone];

        NSDateComponents* startOfWeekComponents = [calendar
                                                   components:NSCalendarUnitCalendar |
                                                   NSCalendarUnitYearForWeekOfYear |
                                                   NSCalendarUnitWeekOfYear
                                                   fromDate:now];
        NSDate* startOfWeek = [startOfWeekComponents date];
        NSDate* endOfWeek = [calendar dateByAddingUnit:NSCalendarUnitDay
                                                 value:7
                                                toDate:startOfWeek
                                               options:0];

        // Convert start and end to ISO 8601 strings
        NSISO8601DateFormatter* isoFormatter = [[NSISO8601DateFormatter alloc] init];
        NSString* viewStart = [isoFormatter stringFromDate:startOfWeek];
        NSString* viewEnd = [isoFormatter stringFromDate:endOfWeek];

        [GraphManager.instance
         getCalendarViewStartingAt:viewStart
         endingAt:viewEnd
         withCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
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

1. <span data-ttu-id="06e40-121">Exécutez l’application, connectez-vous et appuyez sur **l’élément** de navigation Calendrier dans le menu.</span><span class="sxs-lookup"><span data-stu-id="06e40-121">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="06e40-122">Vous devriez voir un vidage JSON des événements dans l’application.</span><span class="sxs-lookup"><span data-stu-id="06e40-122">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="06e40-123">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="06e40-123">Display the results</span></span>

<span data-ttu-id="06e40-124">Vous pouvez désormais remplacer le vidage JSON par un autre qui permet d’afficher les résultats de manière conviviale.</span><span class="sxs-lookup"><span data-stu-id="06e40-124">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="06e40-125">Dans cette section, vous allez modifier la fonction pour renvoyer des objets fortement typés et modifier pour utiliser une vue de `getCalendarViewStartingAt` tableau pour afficher les `CalendarViewController` événements.</span><span class="sxs-lookup"><span data-stu-id="06e40-125">In this section, you will modify the `getCalendarViewStartingAt` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getcalendarviewstartingat"></a><span data-ttu-id="06e40-126">Update getCalendarViewStartingAt</span><span class="sxs-lookup"><span data-stu-id="06e40-126">Update getCalendarViewStartingAt</span></span>

1. <span data-ttu-id="06e40-127">Ouvrez **GraphManager.h**.</span><span class="sxs-lookup"><span data-stu-id="06e40-127">Open **GraphManager.h**.</span></span> <span data-ttu-id="06e40-128">Modifiez `GetCalendarViewCompletionBlock` la définition de type comme suit.</span><span class="sxs-lookup"><span data-stu-id="06e40-128">Change the `GetCalendarViewCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="06e40-129">Ouvrez **GraphManager.m**.</span><span class="sxs-lookup"><span data-stu-id="06e40-129">Open **GraphManager.m**.</span></span> <span data-ttu-id="06e40-130">Remplacez la fonction `getCalendarViewStartingAt` existante par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-130">Replace the existing `getCalendarViewStartingAt` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="06e40-131">Mettre à jour CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="06e40-131">Update CalendarViewController</span></span>

1. <span data-ttu-id="06e40-132">Créez un **fichier de classe Cocoa Touch** dans le projet **GraphTutorial** nommé `CalendarTableViewCell` .</span><span class="sxs-lookup"><span data-stu-id="06e40-132">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="06e40-133">Choisissez **UITableViewCell dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="06e40-133">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06e40-134">Ouvrez **CalendarTableViewCell.h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-134">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="06e40-135">Ouvrez **CalendarTableViewCell.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-135">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="06e40-136">Créez un **fichier de classe Cocoa Touch** dans le projet **GraphTutorial** nommé `CalendarTableViewController` .</span><span class="sxs-lookup"><span data-stu-id="06e40-136">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController`.</span></span> <span data-ttu-id="06e40-137">Choisissez **UITableViewController dans** la **sous-classe du** champ.</span><span class="sxs-lookup"><span data-stu-id="06e40-137">Choose **UITableViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06e40-138">Ouvrez **CalendarTableViewController.h** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-138">Open **CalendarTableViewController.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="06e40-139">Ouvrez **CalendarTableViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-139">Open **CalendarTableViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="06e40-140">Ouvrez **Main.storyboard et** recherchez la **scène de calendrier.**</span><span class="sxs-lookup"><span data-stu-id="06e40-140">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="06e40-141">Supprimez le défilement de l’affichage racine.</span><span class="sxs-lookup"><span data-stu-id="06e40-141">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="06e40-142">À **l’aide de la** bibliothèque, ajoutez une **barre de navigation** en haut de l’affichage.</span><span class="sxs-lookup"><span data-stu-id="06e40-142">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="06e40-143">Double-cliquez sur **le titre dans** la barre de navigation et mettez-le à `Calendar` jour.</span><span class="sxs-lookup"><span data-stu-id="06e40-143">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="06e40-144">À **l’aide de la bibliothèque,** ajoutez un élément de bouton **de** barre à droite de la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="06e40-144">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="06e40-145">Sélectionnez le nouveau bouton de barre, puis l’inspecteur **d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="06e40-145">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06e40-146">Modifier **l’image** **sur plus**.</span><span class="sxs-lookup"><span data-stu-id="06e40-146">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="06e40-147">Ajoutez **un affichage conteneur** de la **bibliothèque** à l’affichage sous la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="06e40-147">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="06e40-148">Resize the container view to take all of the remaining space in the view.</span><span class="sxs-lookup"><span data-stu-id="06e40-148">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="06e40-149">Définissez les contraintes sur la barre de navigation et l’affichage conteneur comme suit.</span><span class="sxs-lookup"><span data-stu-id="06e40-149">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="06e40-150">**Barre de navigation**</span><span class="sxs-lookup"><span data-stu-id="06e40-150">**Navigation Bar**</span></span>
        - <span data-ttu-id="06e40-151">Ajouter une contrainte : Hauteur, valeur : 44</span><span class="sxs-lookup"><span data-stu-id="06e40-151">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="06e40-152">Add constraint: Leading space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-152">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="06e40-153">Add constraint: Trailing space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-153">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="06e40-154">Ajouter une contrainte : espace supérieur à la zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="06e40-154">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="06e40-155">**Affichage conteneur**</span><span class="sxs-lookup"><span data-stu-id="06e40-155">**Container View**</span></span>
        - <span data-ttu-id="06e40-156">Add constraint: Leading space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-156">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="06e40-157">Add constraint: Trailing space to Safe Area, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-157">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="06e40-158">Ajouter une contrainte : espace supérieur dans la barre de navigation en bas, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="06e40-158">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="06e40-159">Ajouter une contrainte : espace inférieur à la zone sécurisée, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="06e40-159">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="06e40-160">Recherchez le deuxième contrôleur d’affichage ajouté au storyboard lorsque vous avez ajouté l’affichage conteneur.</span><span class="sxs-lookup"><span data-stu-id="06e40-160">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="06e40-161">Il est connecté à la **scène de calendrier** par une ségue d’incorporation.</span><span class="sxs-lookup"><span data-stu-id="06e40-161">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="06e40-162">Sélectionnez ce contrôleur et utilisez **l’inspecteur d’identité** pour modifier **la** classe **en CalendarTableViewController**.</span><span class="sxs-lookup"><span data-stu-id="06e40-162">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="06e40-163">Supprimez **l’affichage** du **contrôleur d’affichage de table de calendrier.**</span><span class="sxs-lookup"><span data-stu-id="06e40-163">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="06e40-164">Ajoutez **un affichage Table à** partir de la **bibliothèque** au **contrôleur d’affichage de table de calendrier.**</span><span class="sxs-lookup"><span data-stu-id="06e40-164">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="06e40-165">Sélectionnez l’affichage Tableau, puis **l’inspecteur d’attributs.**</span><span class="sxs-lookup"><span data-stu-id="06e40-165">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06e40-166">Définissez **les cellules prototype** sur **1**.</span><span class="sxs-lookup"><span data-stu-id="06e40-166">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="06e40-167">Faites glisser le bord inférieur de la cellule prototype pour vous donner une zone plus grande à travailler.</span><span class="sxs-lookup"><span data-stu-id="06e40-167">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="06e40-168">Utilisez la **bibliothèque pour** ajouter trois **étiquettes** à la cellule prototype.</span><span class="sxs-lookup"><span data-stu-id="06e40-168">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="06e40-169">Sélectionnez la cellule prototype, puis l’inspecteur **d’identité.**</span><span class="sxs-lookup"><span data-stu-id="06e40-169">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="06e40-170">Change **Class** to **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="06e40-170">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="06e40-171">Sélectionnez **l’inspecteur d’attributs** et définissez **l’identificateur** sur `EventCell` .</span><span class="sxs-lookup"><span data-stu-id="06e40-171">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="06e40-172">Une fois **EventCell** sélectionné, sélectionnez l’Inspecteur de connexions et connectez-vous, ainsi qu’aux étiquettes que vous avez **ajoutées** à la cellule `durationLabel` dans le `organizerLabel` `subjectLabel` storyboard.</span><span class="sxs-lookup"><span data-stu-id="06e40-172">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="06e40-173">Définissez les propriétés et les contraintes sur les trois étiquettes comme suit.</span><span class="sxs-lookup"><span data-stu-id="06e40-173">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="06e40-174">**Étiquette d’objet**</span><span class="sxs-lookup"><span data-stu-id="06e40-174">**Subject Label**</span></span>
        - <span data-ttu-id="06e40-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-176">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-176">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-177">Ajouter une contrainte : espace supérieur à la marge supérieure de l’affichage de contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="06e40-177">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="06e40-178">**Étiquette d’organisateur**</span><span class="sxs-lookup"><span data-stu-id="06e40-178">**Organizer Label**</span></span>
        - <span data-ttu-id="06e40-179">Police : System 12.0</span><span class="sxs-lookup"><span data-stu-id="06e40-179">Font: System 12.0</span></span>
        - <span data-ttu-id="06e40-180">Ajouter une contrainte : Hauteur, valeur : 15</span><span class="sxs-lookup"><span data-stu-id="06e40-180">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="06e40-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-182">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-182">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span><span class="sxs-lookup"><span data-stu-id="06e40-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="06e40-184">**Étiquette de durée**</span><span class="sxs-lookup"><span data-stu-id="06e40-184">**Duration Label**</span></span>
        - <span data-ttu-id="06e40-185">Police : System 12.0</span><span class="sxs-lookup"><span data-stu-id="06e40-185">Font: System 12.0</span></span>
        - <span data-ttu-id="06e40-186">Couleur : gris foncé</span><span class="sxs-lookup"><span data-stu-id="06e40-186">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="06e40-187">Ajouter une contrainte : Hauteur, valeur : 15</span><span class="sxs-lookup"><span data-stu-id="06e40-187">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="06e40-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-189">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span><span class="sxs-lookup"><span data-stu-id="06e40-189">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="06e40-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span><span class="sxs-lookup"><span data-stu-id="06e40-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="06e40-191">Ajouter une contrainte : espace inférieur à la marge inférieure de l’affichage du contenu, valeur : 0</span><span class="sxs-lookup"><span data-stu-id="06e40-191">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="06e40-192">Sélectionnez **EventCell,** puis **l’inspecteur de taille.**</span><span class="sxs-lookup"><span data-stu-id="06e40-192">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="06e40-193">Activer **automatique pour** la hauteur de **ligne**.</span><span class="sxs-lookup"><span data-stu-id="06e40-193">Enable **Automatic** for **Row Height**.</span></span>

    ![Capture d’écran des contrôleurs d’affichage de calendrier et de table de calendrier](images/calendar-table-storyboard.png)

1. <span data-ttu-id="06e40-195">Ouvrez **CalendarViewController.h et** supprimez la `calendarJSON` propriété.</span><span class="sxs-lookup"><span data-stu-id="06e40-195">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>

1. <span data-ttu-id="06e40-196">Ouvrez **CalendarViewController.m** et remplacez son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="06e40-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. <span data-ttu-id="06e40-197">Exécutez l’application, connectez-vous et appuyez sur **l’onglet** Calendrier. Vous devriez voir la liste des événements.</span><span class="sxs-lookup"><span data-stu-id="06e40-197">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/calendar-list.png)
