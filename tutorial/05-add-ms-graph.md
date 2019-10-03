<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser le [Kit de développement logiciel (SDK) Microsoft Graph pour effectuer des](https://github.com/microsoftgraph/msgraph-sdk-objc) appels à Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtenir des événements de calendrier à partir d’Outlook

Dans cette section, vous allez étendre `GraphManager` la classe pour ajouter une fonction afin d’obtenir les événements de l' `CalendarViewController` utilisateur et la mise à jour pour utiliser ces nouvelles fonctions.

1. Ouvrez **GraphManager. h** et ajoutez le code suivant au- `@interface` dessus de la déclaration.

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. Ajoutez le code suivant à la `@interface` déclaration.

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. Ouvrez **GraphManager. m** et ajoutez la fonction suivante à la `GraphManager` classe.

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
    > Examinez le contenu du `getEventsWithCompletionBlock` code.
    >
    > - L’URL qui sera appelée est `/v1.0/me/events`.
    > - Le `select` paramètre de requête limite les champs renvoyés pour chaque événement à seulement ceux que l’application utilisera réellement.
    > - Le `orderby` paramètre de requête trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.

1. Ouvrez **CalendarViewController. m** et remplacez l’intégralité de son contenu par le code suivant.

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

Vous pouvez maintenant exécuter l’application, se connecter et appuyer sur l’élément de navigation **calendrier** dans le menu. Vous devriez voir un dump JSON des événements dans l’application.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez remplacer le vidage JSON par un texte pour afficher les résultats de manière conviviale. Dans cette section, vous allez modifier la `getEventsWithCompletionBlock` fonction pour renvoyer des objets fortement typés et modifier `CalendarViewController` pour qu’elle affiche les événements à l’aide d’un affichage tableau.

### <a name="update-getevents"></a>Mettre à jour getEvents

1. Ouvrez **GraphManager. h**. Modifiez la `GetEventsCompletionBlock` définition de type comme suit.

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. Ouvrez **GraphManager. m**. Remplacez la `completionBlock(data, nil);` ligne dans la `getEventsWithCompletionBlock` fonction par le code suivant.

    ```objc
    NSError* graphError;

    // Deserialize to an events collection
    MSCollection* eventsCollection = [[MSCollection alloc] initWithData:data error:&graphError];
    if (graphError) {
        completionBlock(nil, graphError);
        return;
    }

    // Create an array to return
    NSMutableArray* eventsArray = [[NSMutableArray alloc]
                                initWithCapacity:eventsCollection.value.count];

    for (id event in eventsCollection.value) {
        // Deserialize the event and add to the array
        MSGraphEvent* graphEvent = [[MSGraphEvent alloc] initWithDictionary:event];
        [eventsArray addObject:graphEvent];
    }

    completionBlock(eventsArray, nil);
    ```

### <a name="update-calendarviewcontroller"></a>Mettre à jour CalendarViewController

1. Créez un fichier de **classe Touch de cacao** dans le projet **GraphTutorial** nommé `CalendarTableViewCell`. Choisissez **UITableViewCell** dans la sous- **classe du** champ.
1. Ouvrez **CalendarTableViewCell. h** et remplacez son contenu par le code suivant.

    ```objc
    #import <UIKit/UIKit.h>

    NS_ASSUME_NONNULL_BEGIN

    @interface CalendarTableViewCell : UITableViewCell

    @property (nonatomic) NSString* subject;
    @property (nonatomic) NSString* organizer;
    @property (nonatomic) NSString* duration;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. Ouvrez **CalendarTableViewCell. m** et remplacez son contenu par le code suivant.

    ```objc
    #import "CalendarTableViewCell.h"

    @interface CalendarTableViewCell()

    @property (nonatomic) IBOutlet UILabel *subjectLabel;
    @property (nonatomic) IBOutlet UILabel *organizerLabel;
    @property (nonatomic) IBOutlet UILabel *durationLabel;

    @end

    @implementation CalendarTableViewCell

    - (void)awakeFromNib {
        [super awakeFromNib];
        // Initialization code
    }

    - (void)setSelected:(BOOL)selected animated:(BOOL)animated {
        [super setSelected:selected animated:animated];

        // Configure the view for the selected state
    }

    - (void) setSubject:(NSString *)subject {
        _subject = subject;
        self.subjectLabel.text = subject;
        [self.subjectLabel sizeToFit];
    }

    - (void) setOrganizer:(NSString *)organizer {
        _organizer = organizer;
        self.organizerLabel.text = organizer;
        [self.organizerLabel sizeToFit];
    }

    - (void) setDuration:(NSString *)duration {
        _duration = duration;
        self.durationLabel.text = duration;
        [self.durationLabel sizeToFit];
    }

    @end
    ```

1. Ouvrez **main. Storyboard** et localisez la **scène de calendrier**. Sélectionnez l' **affichage** dans la **scène de calendrier** et supprimez-le.

    ![Capture d’écran de l’affichage dans la scène de calendrier](./images/view-in-calendar-scene.png)

1. Ajoutez un **affichage tableau** de la **bibliothèque** à la **scène calendrier**.
1. Sélectionnez l’affichage tableau, puis sélectionnez l' **inspecteur d’attributs**. Définissez les **cellules prototype** sur **1**.
1. Utilisez la **bibliothèque** pour ajouter trois **étiquettes** à la cellule prototype.
1. Sélectionnez la cellule prototype, puis l' **inspecteur d’identité**. Modifiez **Class** en **CalendarTableViewCell**.
1. Sélectionnez l' **inspecteur d’attributs** et définissez **identificateur** sur `EventCell`.
1. Dans le menu **éditeur** , sélectionnez **résoudre les problèmes de disposition automatique**, puis sélectionnez **Ajouter les contraintes manquantes** sous **toutes les vues dans le contrôleur d’affichage de bienvenue**.
1. Une fois le **EventCell** sélectionné, sélectionnez l' **inspecteur** de connexions `durationLabel`et `organizerLabel`Connectez- `subjectLabel` vous, puis les étiquettes que vous avez ajoutées à la cellule sur la table de montage séquentiel.

    ![Capture d’écran de la mise en page de cellule prototype](./images/prototype-cell-layout.png)

1. Ouvrez **CalendarViewController. h** et supprimez `calendarJSON` la propriété.
1. Modifiez la `@interface` déclaration comme suit.

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. Ouvrez **CalendarViewController. m** et remplacez son contenu par le code suivant.

    ```objc
    #import "WelcomeViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface WelcomeViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation WelcomeViewController

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

    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. Exécutez l’application, connectez-vous, puis appuyez sur l’onglet **calendrier** . La liste des événements doit s’afficher.

    ![Capture d’écran du tableau des événements](./images/calendar-list.png)
