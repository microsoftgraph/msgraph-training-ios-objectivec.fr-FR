<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser le [SDK Microsoft Graph pour l’objectif C](https://github.com/microsoftgraph/msgraph-sdk-objc) pour effectuer des appels à Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

Dans cette section, vous allez étendre la classe pour ajouter une fonction afin d’obtenir les événements de l’utilisateur pour la semaine en cours et mettre à jour pour `GraphManager` `CalendarViewController` utiliser cette nouvelle fonction.

1. Ouvrez **GraphManager.h** et ajoutez le code suivant au-dessus de la `@interface` déclaration.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. Ajoutez le code suivant à la `@interface` déclaration.

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. Ouvrez **GraphManager.m** et ajoutez la fonction suivante à la `GraphManager` classe.

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
    > Réfléchissez à ce que fait `getCalendarViewStartingAt` le code.
    >
    > - L’URL qui sera appelée est `/v1.0/me/calendarview`.
    >   - Les `startDateTime` `endDateTime` paramètres de requête définissent le début et la fin de l’affichage Calendrier.
    >   - Le paramètre de requête limite les champs renvoyés pour chaque événement à ceux que `select` l’affichage utilisera réellement.
    >   - Le `orderby` paramètre de requête trie les résultats par heure de début.
    >   - Le `top` paramètre de requête demande 25 résultats par page.
    >   - L’en-tête indique à Microsoft Graph de renvoyer les heures de début et de fin de chaque événement dans le `Prefer: outlook.timezone` fuseau horaire de l’utilisateur.

1. Créez une **classe Cocoa Touch dans** le projet **GraphTutorial** nommé **GraphToIana**. Choisissez **NSObject dans** la **sous-classe du** champ.
1. Ouvrez **GraphToIana.h** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. Ouvrez **GraphToIana.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    Cette opération permet de rechercher simplement un identificateur de fuseau horaire IANA basé sur le nom de fuseau horaire renvoyé par Microsoft Graph.

1. Ouvrez **CalendarViewController.m** et remplacez tout son contenu par le code suivant.

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

1. Exécutez l’application, connectez-vous et appuyez sur **l’élément** de navigation Calendrier dans le menu. Vous devriez voir un vidage JSON des événements dans l’application.

## <a name="display-the-results"></a>Afficher les résultats

Vous pouvez désormais remplacer le vidage JSON par un autre qui permet d’afficher les résultats de manière conviviale. Dans cette section, vous allez modifier la fonction pour renvoyer des objets fortement typés et modifier pour utiliser une vue de `getCalendarViewStartingAt` tableau pour afficher les `CalendarViewController` événements.

### <a name="update-getcalendarviewstartingat"></a>Update getCalendarViewStartingAt

1. Ouvrez **GraphManager.h**. Modifiez `GetCalendarViewCompletionBlock` la définition de type comme suit.

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Ouvrez **GraphManager.m**. Remplacez la fonction `getCalendarViewStartingAt` existante par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a>Mettre à jour CalendarViewController

1. Créez un **fichier de classe Cocoa Touch** dans le projet **GraphTutorial** nommé `CalendarTableViewCell` . Choisissez **UITableViewCell dans** la **sous-classe du** champ.
1. Ouvrez **CalendarTableViewCell.h** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. Ouvrez **CalendarTableViewCell.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. Créez un **fichier de classe Cocoa Touch** dans le projet **GraphTutorial** nommé `CalendarTableViewController` . Choisissez **UITableViewController dans** la **sous-classe du** champ.
1. Ouvrez **CalendarTableViewController.h** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. Ouvrez **CalendarTableViewController.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. Ouvrez **Main.storyboard et** recherchez la **scène de calendrier.** Supprimez le défilement de l’affichage racine.
1. À **l’aide de la** bibliothèque, ajoutez une **barre de navigation** en haut de l’affichage.
1. Double-cliquez sur **le titre dans** la barre de navigation et mettez-le à `Calendar` jour.
1. À **l’aide de la bibliothèque,** ajoutez un élément de bouton **de** barre à droite de la barre de navigation.
1. Sélectionnez le nouveau bouton de barre, puis l’inspecteur **d’attributs.** Modifier **l’image** **sur plus**.
1. Ajoutez **un affichage conteneur** de la **bibliothèque** à l’affichage sous la barre de navigation. Resize the container view to take all of the remaining space in the view.
1. Définissez les contraintes sur la barre de navigation et l’affichage conteneur comme suit.

    - **Barre de navigation**
        - Ajouter une contrainte : Hauteur, valeur : 44
        - Add constraint: Leading space to Safe Area, value: 0
        - Add constraint: Trailing space to Safe Area, value: 0
        - Ajouter une contrainte : espace supérieur à la zone sécurisée, valeur : 0
    - **Affichage conteneur**
        - Add constraint: Leading space to Safe Area, value: 0
        - Add constraint: Trailing space to Safe Area, value: 0
        - Ajouter une contrainte : espace supérieur dans la barre de navigation en bas, valeur : 0
        - Ajouter une contrainte : espace inférieur à la zone sécurisée, valeur : 0

1. Recherchez le deuxième contrôleur d’affichage ajouté au storyboard lorsque vous avez ajouté l’affichage conteneur. Il est connecté à la **scène de calendrier** par une ségue d’incorporation. Sélectionnez ce contrôleur et utilisez **l’inspecteur d’identité** pour modifier **la** classe **en CalendarTableViewController**.
1. Supprimez **l’affichage** du **contrôleur d’affichage de table de calendrier.**
1. Ajoutez **un affichage Table à** partir de la **bibliothèque** au **contrôleur d’affichage de table de calendrier.**
1. Sélectionnez l’affichage Tableau, puis **l’inspecteur d’attributs.** Définissez **les cellules prototype** sur **1**.
1. Faites glisser le bord inférieur de la cellule prototype pour vous donner une zone plus grande à travailler.
1. Utilisez la **bibliothèque pour** ajouter trois **étiquettes** à la cellule prototype.
1. Sélectionnez la cellule prototype, puis l’inspecteur **d’identité.** Change **Class** to **CalendarTableViewCell**.
1. Sélectionnez **l’inspecteur d’attributs** et définissez **l’identificateur** sur `EventCell` .
1. Une fois **EventCell** sélectionné, sélectionnez l’Inspecteur de connexions et connectez-vous, ainsi qu’aux étiquettes que vous avez **ajoutées** à la cellule `durationLabel` dans le `organizerLabel` `subjectLabel` storyboard.
1. Définissez les propriétés et les contraintes sur les trois étiquettes comme suit.

    - **Étiquette d’objet**
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Add constraint: Trailing space to Content View Trailing Margin, value: 0
        - Ajouter une contrainte : espace supérieur à la marge supérieure de l’affichage de contenu, valeur : 0
    - **Étiquette d’organisateur**
        - Police : System 12.0
        - Ajouter une contrainte : Hauteur, valeur : 15
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Add constraint: Trailing space to Content View Trailing Margin, value: 0
        - Add constraint: Top space to Subject Label Bottom, value: Standard
    - **Étiquette de durée**
        - Police : System 12.0
        - Couleur : gris foncé
        - Ajouter une contrainte : Hauteur, valeur : 15
        - Add constraint: Leading space to Content View Leading Margin, value: 0
        - Add constraint: Trailing space to Content View Trailing Margin, value: 0
        - Add constraint: Top space to Organizer Label Bottom, value: Standard
        - Ajouter une contrainte : espace inférieur à la marge inférieure de l’affichage du contenu, valeur : 0

1. Sélectionnez **EventCell,** puis **l’inspecteur de taille.** Activer **automatique pour** la hauteur de **ligne**.

    ![Capture d’écran des contrôleurs d’affichage de calendrier et de table de calendrier](images/calendar-table-storyboard.png)

1. Ouvrez **CalendarViewController.h et** supprimez la `calendarJSON` propriété.

1. Ouvrez **CalendarViewController.m** et remplacez son contenu par le code suivant.

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. Exécutez l’application, connectez-vous et appuyez sur **l’onglet** Calendrier. Vous devriez voir la liste des événements.

    ![Capture d’écran du tableau des événements](./images/calendar-list.png)
