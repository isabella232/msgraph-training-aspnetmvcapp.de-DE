<!-- markdownlint-disable MD002 MD041 -->

Öffnen Sie Visual Studio, und wählen Sie **Datei #a0 neues #a1 Projekt**aus. Führen Sie im Dialogfeld **Neues Projekt** folgende Schritte aus:

1. Wählen Sie **Vorlagen #a0 Visual C# #a1-Webs**aus.
1. Wählen Sie **ASP.NET-Webanwendung (.NET Framework)** aus.
1. Geben Sie **Graph-Tutorial** für den Namen des Projekts ein.

    ![Visual Studio 2017 Dialogfeld "Neues Projekt erstellen"](./images/vs-new-project-01.png)

    > [!NOTE]
    > Stellen Sie sicher, dass Sie genau den gleichen Namen für das Visual Studio Projekt eingeben, das in diesen Übungseinheiten angegeben ist. Der Visual Studio Projektname wird Teil des Namespaces im Code. Der Code in diesen Anweisungen hängt vom Namespace ab, der dem in diesen Anweisungen angegebenen Visual Studio Projektnamen entspricht. Wenn Sie einen anderen Projektnamen verwenden, wird der Code nur dann kompiliert, wenn Sie alle Namespaces so anpassen, dass Sie dem Visual Studio Projektnamen entsprechen, den Sie beim Erstellen des Projekts eingeben.

1. Wählen Sie **OK** aus. Wählen Sie im Dialogfeld **neue ASP.NET-Webanwendungsprojekt** die Option **MVC** (unter **ASP.NET 4.7.2 Templates**) aus, und klicken Sie auf **OK**.

1. Drücken Sie **F5** , oder wählen Sie **Debug #a0 Debuggen starten**aus. Wenn alles funktioniert, sollte der Standardbrowser geöffnet und eine standardmäßige ASP.NET-Seite angezeigt werden.

## <a name="add-nuget-packages"></a>NuGet-Pakete hinzufügen

Bevor Sie fortfahren, aktualisieren `bootstrap` Sie das NuGet-Paket, und installieren Sie einige zusätzliche NuGet-Pakete, die Sie später verwenden werden.

- [Microsoft. Owin. Host. SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb/) zum Aktivieren der [Owin](http://owin.org/) -Schnittstellen in der ASP.NET-Anwendung.
- [Microsoft. Owin. Security. OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) für die Durchführung der OpenID Connect-Authentifizierung mit Azure.
- [Microsoft. Owin. Security. Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies/) , um die Cookie-basierte Authentifizierung zu aktivieren.
- [Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) zum Anfordern und Verwalten von Zugriffstoken.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) für das tätigen von Anrufen an Microsoft Graph.

1. Wählen Sie **Tools #a0 NuGet-Paket-Manager #a1-Paket-Manager-Konsole**aus.
1. Geben Sie in der Paket-Manager-Konsole die folgenden Befehle ein.

    ```Powershell
    Update-Package bootstrap
    Install-Package Microsoft.Owin.Host.SystemWeb
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Identity.Client -Version 3.0.8
    Install-Package Microsoft.Graph -Version 1.15.0
    ```

## <a name="design-the-app"></a>Entwerfen der APP

In diesem Abschnitt wird die grundlegende Struktur der Anwendung erstellt.

1. Erstellen Sie eine grundlegende OWIN-Startklasse. Klicken Sie im Projekt `graph-tutorial` Mappen-Explorer mit der rechten Maustaste auf den Ordner, und wählen Sie **#a0 neues Element hinzufügen**. Wählen Sie die **OWIN-Startklassen** Vorlage aus, `Startup.cs`benennen Sie die Datei, und wählen Sie **Hinzufügen**aus.

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf den Ordner **Modelle** , und wählen Sie **#a0 Klasse hinzufügen**aus. Nennen Sie die `Alert` Klasse, und wählen Sie **Hinzufügen**aus. Fügen Sie den folgenden Code `Alert.cs`in hinzu. Sie verwenden diese Klasse, um Fehlermeldungen in den App-Ansichten zu flashen.

    ```cs
    namespace graph_tutorial.Models
    {
        public class Alert
        {
            public const string AlertKey = "TempDataAlerts";
            public string Message { get; set; }
            public string Debug { get; set; }
        }
    }
    ```

1. Öffnen Sie `./Views/Shared/_Layout.cshtml` die Datei, und ersetzen Sie den gesamten Inhalt durch den folgenden Code, um das globale Layout der APP zu aktualisieren.

    ```html
    @{
        var alerts = TempData.ContainsKey(graph_tutorial.Models.Alert.AlertKey) ?
            (List<graph_tutorial.Models.Alert>)TempData[graph_tutorial.Models.Alert.AlertKey] :
            new List<graph_tutorial.Models.Alert>();
    }

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>ASP.NET Graph Tutorial</title>
        @Styles.Render("~/Content/css")
        @Scripts.Render("~/bundles/modernizr")

        <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
              integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt"
              crossorigin="anonymous">
    </head>

    <body>
        <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
            <div class="container">
                @Html.ActionLink("ASP.NET Graph Tutorial", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
                        aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarCollapse">
                    <ul class="navbar-nav mr-auto">
                        <li class="nav-item">
                            @Html.ActionLink("Home", "Index", "Home", new { area = "" },
                                new { @class = ViewBag.Current == "Home" ? "nav-link active" : "nav-link" })
                        </li>
                        @if (Request.IsAuthenticated)
                        {
                            <li class="nav-item" data-turbolinks="false">
                                @Html.ActionLink("Calendar", "Index", "Calendar", new { area = "" },
                                    new { @class = ViewBag.Current == "Calendar" ? "nav-link active" : "nav-link" })
                            </li>
                        }
                    </ul>
                    <ul class="navbar-nav justify-content-end">
                        <li class="nav-item">
                            <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                                <i class="fas fa-external-link-alt mr-1"></i>Docs
                            </a>
                        </li>
                        @if (Request.IsAuthenticated)
                        {
                            <li class="nav-item dropdown">
                                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                                    @if (!string.IsNullOrEmpty(ViewBag.User.Avatar))
                                    {
                                        <img src="@ViewBag.User.Avatar" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                                    }
                                    else
                                    {
                                        <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                                    }
                                </a>
                                <div class="dropdown-menu dropdown-menu-right">
                                    <h5 class="dropdown-item-text mb-0">@ViewBag.User.DisplayName</h5>
                                    <p class="dropdown-item-text text-muted mb-0">@ViewBag.User.Email</p>
                                    <div class="dropdown-divider"></div>
                                    @Html.ActionLink("Sign Out", "SignOut", "Account", new { area = "" }, new { @class = "dropdown-item" })
                                </div>
                            </li>
                        }
                        else
                        {
                            <li class="nav-item">
                                @Html.ActionLink("Sign In", "SignIn", "Account", new { area = "" }, new { @class = "nav-link" })
                            </li>
                        }
                    </ul>
                </div>
            </div>
        </nav>
        <main role="main" class="container">
            @foreach (var alert in alerts)
            {
                <div class="alert alert-danger" role="alert">
                    <p class="mb-3">@alert.Message</p>
                    @if (!string.IsNullOrEmpty(alert.Debug))
                    {
                        <pre class="alert-pre border bg-light p-2"><code>@alert.Debug</code></pre>
                    }
                </div>
            }

            @RenderBody()
        </main>
        @Scripts.Render("~/bundles/jquery")
        @Scripts.Render("~/bundles/bootstrap")
        @RenderSection("scripts", required: false)
    </body>
    </html>
    ```

    Dieser Code fügt [Bootstrap](https://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert und die `Alert` Klasse verwendet, um Warnungen anzuzeigen.

1. Öffnen `Content/Site.css` Sie den gesamten Inhalt, und ersetzen Sie ihn durch den folgenden Code.

    ```css
    body {
      padding-top: 4.5rem;
    }

    .alert-pre {
      word-wrap: break-word;
      word-break: break-all;
      white-space: pre-wrap;
    }
    ```

1. Öffnen Sie `Views/Home/index.cshtml` die Datei, und ersetzen Sie den Inhalt durch Folgendes.

    ```html
    @{
        ViewBag.Current = "Home";
    }

    <div class="jumbotron">
        <h1>ASP.NET Graph Tutorial</h1>
        <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from ASP.NET</p>
        @if (Request.IsAuthenticated)
        {
            <h4>Welcome @ViewBag.User.DisplayName!</h4>
            <p>Use the navigation bar at the top of the page to get started.</p>
        }
        else
        {
            @Html.ActionLink("Click here to sign in", "SignIn", "Account", new { area = "" }, new { @class = "btn btn-primary btn-large" })
        }
    </div>
    ```

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf den Ordner **Controller** , und wählen Sie **#a0 Controller hinzufügen**aus.... Wählen Sie **MVC 5 Controller – leer** , und wählen Sie **Hinzufügen**aus. Nennen Sie den `BaseController` Controller, und wählen Sie **Hinzufügen**aus. Ersetzen Sie den Inhalt von `BaseController.cs` durch den folgenden Code.

    ```cs
    using graph_tutorial.Models;
    using System.Collections.Generic;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public abstract class BaseController : Controller
        {
            protected void Flash(string message, string debug=null)
            {
                var alerts = TempData.ContainsKey(Alert.AlertKey) ?
                    (List<Alert>)TempData[Alert.AlertKey] :
                    new List<Alert>();

                alerts.Add(new Alert
                {
                    Message = message,
                    Debug = debug
                });

                TempData[Alert.AlertKey] = alerts;
            }
        }
    }
    ```

    Jeder Controller kann von dieser Basis Controllerklasse erben, um Zugriff auf die `Flash` Funktion zu erhalten. Aktualisieren Sie `HomeController` die-Klasse, `BaseController`um von zu erben.

1. Öffnen `Controllers/HomeController.cs` Sie und ändern `public class HomeController : Controller` Sie die-Reihe in:

    ```cs
    public class HomeController : BaseController
    ```

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

    ![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)