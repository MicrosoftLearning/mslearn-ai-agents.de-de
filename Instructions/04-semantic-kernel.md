---
lab:
  title: Entwickeln von Azure KI-Agents mit dem Semantischer-Kernel-SDK
  description: 'Erfahren Sie, wie Sie das Semantischer-Kernel-SDK verwenden, um einen Agent des Azure AI Agent-Dienstes zu erstellen und zu verwenden.'
---

# Entwickeln von Azure KI-Agents mit dem Semantischer-Kernel-SDK

In dieser Übung verwenden Sie den Azure AI Agent-dienst und den semantischen Kernel, um einen KI-Agent zu erstellen, der Spesenforderungen verarbeitet.

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Einige der in dieser Übung verwendeten Technologien befinden sich in der Vorschau oder in der aktiven Entwicklung. Es kann zu unerwartetem Verhalten, Warnungen oder Fehlern kommen.

## Bereitstellen Sie ein DALL-E-Modell in einem Azure KI Foundry-Projekt

Beginnen wir mit dem Erstellen eines Azure KI Foundry-Projekts.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipps oder Schnellstartfenster, die bei der ersten Anmeldung geöffnet werden, und verwenden Sie gegebenenfalls das Logo **Azure AI Foundry** oben links, um zur Startseite zu navigieren, die ähnlich wie die folgende Abbildung aussieht (schließen Sie das **Hilfe**-Fenster, falls es geöffnet ist):

    ![Screenshot des Azure KI Foundry-Portals.](./Media/ai-foundry-home.png)

1. Suchen Sie auf der Startseite im Abschnitt **Modelle und Funktionen erkunden** nach dem Modell `gpt-4o`, das wir in unserem Projekt verwenden werden.
1. Wählen Sie in den Suchergebnissen das Modell **gpt-4o** aus, um dessen Details anzuzeigen, und wählen Sie dann oben auf der Seite für das Modell die Option **Dieses Modell verwenden** aus.
1. Wenn Sie zum Erstellen eines Projekts aufgefordert werden, geben Sie einen gültigen Namen für Ihr Projekt ein und erweitern Sie **Erweiterte Optionen**.
1. Bestätigen Sie die folgenden Einstellungen für Ihr Projekt:
    - **Azure KI Foundry-Ressource**: *Ein gültiger Name für Ihre Azure KI Foundry-Ressource*
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Region**: *Wählen Sie einen beliebigen Standort aus, an dem KI Services unterstützt wird***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Wählen Sie **Erstellen** und warten Sie, bis Ihr Projekt einschließlich der von Ihnen ausgewählten GPT-4-Modellbereitstellung erstellt wurde.
1. Wenn Ihr Projekt erstellt wird, wird der Chat-Playground automatisch geöffnet.
1. Notieren Sie sich im Bereich **Setup** den Namen Ihrer Modellbereitstellung, der **gpt-4o** lauten sollte. Sie können dies überprüfen, indem Sie die Bereitstellung auf der Seite **Modelle und Endpunkte** anzeigen (öffnen Sie dazu einfach diese Seite im Navigationsbereich auf der linken Seite).
1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    > **Hinweis**: Wenn die Fehlermeldung *Unzureichende Berechtigungen** angezeigt wird, klicken Sie auf die Schaltfläche „**Beheben**“, um das Problem zu beheben.

    ![Screenshot eines Azure KI-Projekts im Azure AI Foundry-Portal.](./Media/ai-foundry-project.png)

## Erstellen einer Agent-Client-App 

Jetzt können Sie eine Client-App erstellen, die einen Agent sowie eine benutzerdefinierte Funktion definiert. Ein Teil des Codes wurde für Sie in einem GitHub-Repository bereitgestellt.

### Vorbereiten der Umgebung

1. Öffnen Sie eine neue Browserregisterkarte (wobei das Azure AI Foundry-Portal auf der vorhandenen Registerkarte geöffnet bleibt). Wechseln Sie dann in der neuen Registerkarte zum [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldedaten an, wenn Sie dazu aufgefordert werden.

    Schließen Sie alle Willkommensbenachrichtigungen, um die Startseite des Azure-Portals anzuzeigen.

1. Verwenden Sie die Schaltfläche **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud-Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung ohne Speicher in Ihrem Abonnement.

    Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Fenster am unteren Rand des Azure-Portals. Sie können die Größe dieses Bereichs ändern oder maximieren, um die Arbeit zu vereinfachen.

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** das Menüelement **Zur klassischen Version wechseln** aus (dies ist für die Verwendung des Code-Editors erforderlich).

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im Cloud Shell-Bereich die folgenden Befehle ein, um das GitHub-Repository mit den Codedateien für diese Übung zu klonen (geben Sie den Befehl ein oder kopieren Sie ihn in die Zwischenablage und klicken Sie dann mit der rechten Maustaste in die Befehlszeile, um ihn als reinen Text einzufügen):

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **Tipp**: Wenn Sie Befehle in die Cloud Shell einfügen, kann die Ausgabe einen großen Teil des Bildschirmpuffers in Anspruch nehmen und der Cursor in der aktuellen Zeile kann verdeckt sein. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

1. Wenn das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Arbeitsverzeichnis in den Ordner mit den Codedateien zu ändern und alle aufzulisten.

    ```
   cd ai-agents/Labfiles/04-semantic-kernel/python
   ls -a -l
    ```

    Die bereitgestellten Dateien enthalten Anwendungscode für Konfigurationseinstellungen und eine Datei mit Spesendaten.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Hinweis**: Durch die Installation von *semantic-kernel[azure]* wird eine mit dem semantischen Kernel kompatible Version von *azure-ai-projects* installiert.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Code-Datei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrer gpt-4o-Modellbereitstellung zugewiesen haben.
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Schreiben von Code für eine Agent-App.

> **Tipp**: Achten Sie beim Hinzufügen von Code auf den richtigen Einzug. Verwenden Sie die vorhandenen Kommentare als Leitfaden, und geben Sie den neuen Code auf derselben Einzugsebene ein.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Agent-Code-Datei zu bearbeiten:

    ```
   code semantic-kernel.py
    ```

1. Sehen Sie sich den Code in der Datei an. Sie enthält folgende Elemente:
    - Einige **Importanweisungen** zum Hinzufügen von Verweisen auf häufig verwendete Namespaces
    - Eine *Hauptfunktion*, die eine Datei mit Spesendaten lädt, den Benutzer nach Anweisungen fragt und dann aufruft ...
    - Eine **process_expenses_data**-Funktion, in der der Code zum Erstellen und Verwenden des Agents hinzugefügt werden muss
    - Eine **EmailPlugin**-Klasse, die eine Kernelfunktion mit dem Namen **send_email** enthält. Diese wird von Ihrem Agent verwendet, um die Funktionalität zum Senden einer E-Mail zu simulieren.

1. Suchen Sie oben in der Datei nach der vorhandenen **Importanweisungen**, suchen Sie den Kommentar **Verweise hinzufügen**, und fügen Sie den folgenden Code hinzu, um auf die Namespaces in den Bibliotheken zu verweisen, die Sie zum Implementieren Ihres Agents benötigen:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. Suchen Sie unten in der Datei den Kommentar **Erstellen eines Plug-Ins für die E-Mail-Funktionalität**, und fügen Sie den folgenden Code hinzu, um eine Klasse für ein Plug-In zu definieren, das eine Funktion enthält, die Ihr Agent zum Senden von E-Mails nutzen wird (Plug-Ins sind eine Möglichkeit, benutzerdefinierte Funktionen zu Semantischer-Kernel-Agents hinzuzufügen).

    ```python
   # Create a Plugin for the email functionality
   class EmailPlugin:
       """A Plugin to simulate email functionality."""
    
       @kernel_function(description="Sends an email.")
       def send_email(self,
                      to: Annotated[str, "Who to send the email to"],
                      subject: Annotated[str, "The subject of the email."],
                      body: Annotated[str, "The text body of the email."]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **Hinweis**: Die Funktion *simuliert* das Senden einer E-Mail durch Drucken an die Konsole. In einer echten Anwendung verwenden Sie einen SMTP-Dienst oder ähnliches, um die E-Mail tatsächlich zu senden!

1. Sichern Sie oben den neuen **EmailPlugin**-Klassencode in der Funktion **create_expense_claim**, suchen Sie den Kommentar **Konfigurationseinstellungen abrufen**, und fügen Sie den folgenden Code hinzu, um die Konfigurationsdatei zu laden und ein Objekt **AzureAIAgentSettings** zu erstellen (das automatisch die Azure KI Agent-Einstellungen aus der Konfiguration enthält).

    (Achten Sie darauf, die Ebene des Einzugs beizubehalten)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. Suchen Sie den Kommentar **Verbindung mit dem Azure AI Foundry-Projekt**, und fügen Sie den folgenden Code hinzu, um sich mit Ihrem Azure AI Foundry-Projekt zu verbinden. Verwenden Sie dazu die Azure-Anmeldeinformationen, mit denen Sie derzeit angemeldet sind.

    (Achten Sie darauf, die Ebene des Einzugs beizubehalten)

    ```python
   # Connect to the Azure AI Foundry project
   async with (
        DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(
            credential=creds
        ) as project_client,
   ):
    ```

1. Suchen Sie den Kommentar **Definieren eines Azure KI-Agents, der eine Spesenabrechnungs-E-Mail sendet**, und fügen Sie den folgenden Code hinzu, um eine Azure KI-Agent-Definition für Ihren Agent zu erstellen.

    (Achten Sie darauf, die Ebene des Einzugs beizubehalten)

    ```python
   # Define an Azure AI agent that sends an expense claim email
   expenses_agent_def = await project_client.agents.create_agent(
        model= ai_agent_settings.model_deployment_name,
        name="expenses_agent",
        instructions="""You are an AI assistant for expense claim submission.
                        When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                        Then confirm to the user that you've done so."""
   )
    ```

1. Suchen Sie den Kommentar **Erstellen eines Semantischer-Kernel-Agents**, und fügen Sie den folgenden Code hinzu, um ein Semantischer-Kernel-Agent-Objekt für Ihren Azure KI-Agent zu erstellen, der einen Verweis auf das **EmailPlugin**-Plug-In enthält.

    (Achten Sie darauf, die Einzugsebene beizubehalten)

    ```python
   # Create a semantic kernel agent
   expenses_agent = AzureAIAgent(
        client=project_client,
        definition=expenses_agent_def,
        plugins=[EmailPlugin()]
   )
    ```

1. Suchen Sie den Kommentar **Verwenden des Agents, um die Spesendaten zu verarbeiten**, und fügen Sie den folgenden Code hinzu, um einen Thread zu erstellen, in dem Ihr Agent ausgeführt werden soll, und rufen Sie ihn dann mit einer Chatnachricht auf.

    (Achten Sie darauf, die Einzugsebene beizubehalten):

    ```python
   # Use the agent to process the expenses data
   thread: AzureAIAgentThread = AzureAIAgentThread(client=project_client)
   try:
        # Add the input prompt to a list of messages to be submitted
        prompt_messages = [f"{prompt}: {expenses_data}"]
        # Invoke the agent for the specified thread with the messages
        response = await expenses_agent.get_response(thread_id=thread.id, messages=prompt_messages)
        # Display the response
        print(f"\n# {response.name}:\n{response}")
   except Exception as e:
        # Something went wrong
        print (e)
   finally:
        # Cleanup: Delete the thread and agent
        await thread.delete() if thread else None
        await project_client.agents.delete_agent(expenses_agent.id)
    ```

1. Überprüfen Sie, ob der abgeschlossene Code für Ihren Agent mithilfe der Kommentare verstehen kann, was jeder Codeblock bewirkt, und speichern Sie dann Die Codeänderungen (**STRG+S**).
1. Lassen Sie den Code-Editor geöffnet, falls Sie die Eingabefehler im Code korrigieren müssen, aber ändern Sie die Größe der Bereiche, damit Sie mehr von der Befehlszeilenkonsole sehen können.

### Bei Azure anmelden und die App ausführen

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell unterhalb des Code-Editors den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
    az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry Hub enthält, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
   python semantic-kernel.py
    ```
    
    Die Anwendung wird mit den Anmeldeinformationen für Ihre authentifizierte Azure-Sitzung ausgeführt, um eine Verbindung mit Ihrem Projekt herzustellen und den Agent zu erstellen und auszuführen.

1. Wenn Sie gefragt werden, was mit den Spesendaten zu tun ist, geben Sie den folgenden Prompt ein:

    ```
   Submit an expense claim
    ```

1. Überprüfe nach Abschluss der Anwendung die Ausgabe. Der Agent sollte eine E-Mail für eine Spesenabrechnung basierend auf den bereitgestellten Daten erstellt haben.

    > **Tipp**: Wenn die App fehlschlägt, weil das Ratenlimit überschritten wird. Warten Sie einige Sekunden, und versuchen Sie es noch mal. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

## Zusammenfassung

In dieser Übung haben Sie das Azure AI Agent Service SDK und den semantischen Kernel verwendet, um einen Agent zu erstellen.

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
