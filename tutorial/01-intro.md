<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="79b09-101">Ce didacticiel vous apprend à créer une application React Native qui utilise l’API Microsoft Graph pour récupérer les informations de calendrier d’un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="79b09-101">This tutorial teaches you how to build an React Native app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="79b09-102">Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez télécharger ou cloner le référentiel [GitHub.](https://github.com/microsoftgraph/msgraph-training-ios-objectivec)</span><span class="sxs-lookup"><span data-stu-id="79b09-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="79b09-103">Configuration requise</span><span class="sxs-lookup"><span data-stu-id="79b09-103">Prerequisites</span></span>

<span data-ttu-id="79b09-104">Avant de commencer ce didacticiel, les instructions suivantes doivent être installées sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="79b09-104">Before you start this tutorial, you should have the following installed on your development machine.</span></span>

- [<span data-ttu-id="79b09-105">Xcode</span><span class="sxs-lookup"><span data-stu-id="79b09-105">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="79b09-106">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="79b09-106">CocoaPods</span></span>](https://cocoapods.org)

<span data-ttu-id="79b09-107">Vous devez également avoir un compte Microsoft personnel avec une boîte aux lettres sur Outlook.com ou un compte scolaire ou scolaire Microsoft.</span><span class="sxs-lookup"><span data-stu-id="79b09-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="79b09-108">Si vous n’avez pas de compte Microsoft, deux options s’offrent à vous pour obtenir un compte gratuit :</span><span class="sxs-lookup"><span data-stu-id="79b09-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="79b09-109">Vous pouvez [vous inscrire à un nouveau compte Microsoft personnel.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="79b09-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="79b09-110">Vous pouvez vous inscrire au programme pour les développeurs [Office 365](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement Office 365 gratuit.</span><span class="sxs-lookup"><span data-stu-id="79b09-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="79b09-111">Ce didacticiel a été écrit à l’aide de Xcode version 12.3 et CocoaPods version 1.10.1.</span><span class="sxs-lookup"><span data-stu-id="79b09-111">This tutorial was written using Xcode version 12.3 and CocoaPods version 1.10.1.</span></span> <span data-ttu-id="79b09-112">Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais elles n’ont pas été testées.</span><span class="sxs-lookup"><span data-stu-id="79b09-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="79b09-113">Commentaires</span><span class="sxs-lookup"><span data-stu-id="79b09-113">Feedback</span></span>

<span data-ttu-id="79b09-114">Veuillez nous faire part de vos commentaires sur ce didacticiel dans [le référentiel GitHub.](https://github.com/microsoftgraph/msgraph-training-ios-objectivec)</span><span class="sxs-lookup"><span data-stu-id="79b09-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-objectivec).</span></span>
