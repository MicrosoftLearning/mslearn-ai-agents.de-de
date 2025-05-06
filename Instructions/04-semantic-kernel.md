---
lab:
  title: Entwickeln von Azure KI-Agents mit dem Semantischer-Kernel-SDK
  description: 'Erfahren Sie, wie Sie das Semantischer-Kernel-SDK verwenden, um einen Azure AI Agent-Dienst-Agent zu erstellen und zu verwenden.'
---

# Entwickeln von Azure KI-Agents mit dem Semantischer-Kernel-SDK

In dieser Übung verwenden Sie den Azure AI Agent-dienst und den semantischen Kernel, um einen KI-Agent zu erstellen, der Spesenforderungen verarbeitet.

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Einige der in dieser Übung verwendeten Technologien befinden sich in der Vorschau oder in der aktiven Entwicklung. Es kann zu unerwartetem Verhalten, Warnungen oder Fehlern kommen.

## Erstellen eines Azure KI Foundry-Projekts

Beginnen wir mit dem Erstellen eines Azure AI Foundry-Projekts.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipps oder Schnellstartfenster, die bei der ersten Anmeldung geöffnet werden, und verwenden Sie gegebenenfalls das Logo **Azure AI Foundry** oben links, um zur Startseite zu navigieren, die ähnlich wie die folgende Abbildung aussieht (schließen Sie das **Hilfe**-Fenster, falls es geöffnet ist):

    ![Screenshot des Azure KI Foundry-Portals.](./Media/ai-foundry-home.png)

1. Wählen Sie auf der Startseite **+ Projekt erstellen**.
1. Geben Sie im Assistenten **Projekt erstellen** einen gültigen Namen für Ihr Projekt ein und wählen Sie, falls ein vorhandener Hub vorgeschlagen wird, die Option, einen neuen zu erstellen. Überprüfen Sie dann die Azure-Ressourcen, die automatisch erstellt werden, um Ihren Hub und Ihr Projekt zu unterstützen.
1. Wählen Sie **Anpassen** aus und legen Sie die folgenden Einstellungen für Ihren Hub fest:
    - **Hubname**: *Geben Sie einen gültigen Namen für Ihren Hub an*
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Standort:** Wählen Sie eine der folgenden Regionen aus\*:
        - eastus
        - eastus2
        - swedencentral
        - westus
        - westus3
    - A**zure KI Services oder Azure OpenAI verbinden**: *Wählen Sie Neuen KI-Dienst erstellen aus*
    - **Azure KI-Suche verbinden**: Verbindung überspringen

    > \* Zum Zeitpunkt des Schreibens unterstützen diese Regionen das gpt-4o-Modell für die Verwendung in Agents. Die Modellverfügbarkeit wird durch regionale Kontingente eingeschränkt. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze erreicht werden, besteht die Möglichkeit, dass Sie ein weiteres Projekt in einer anderen Region erstellen müssen.

1. Klicken Sie auf **Weiter**, um Ihre Konfiguration zu überprüfen. Klicken Sie auf **Erstellen** und warten Sie, bis der Vorgang abgeschlossen ist.
1. Sobald Ihr Projekt erstellt wurde, schließen Sie alle angezeigten Tipps und überprüfen Sie die Projektseite im Azure AI Foundry-Portal, die in etwa wie in der folgenden Abbildung aussehen sollte:

    ![Screenshot eines Azure KI-Projekts im Azure AI Foundry-Portal.](./Media/ai-foundry-project.png)

## Bereitstellen eines generativen KI-Modells

Jetzt können Sie ein generatives KI-Sprachmodell bereitstellen, um Ihren Agent zu unterstützen.

1. Wählen Sie im linken Fensterbereich für Ihr Projekt im Abschnitt **Meine Assets** die Seite **Modelle + Endpunkte**.
1. Wählen Sie auf der Seite **Modelle + Endpunkte** auf der Registerkarte **Modellbereitstellungen** im Menü **+ Modell bereitstellen** die Option **Basismodell bereitstellen**.
1. Suchen Sie das Modell **gpt-4o** in der Liste, wählen Sie es aus und bestätigen Sie es.
1. Stellen Sie das Modell mit den folgenden Einstellungen bereit, indem Sie **Anpassen** in den Bereitstellungsdetails wählen:
    - **Bereitstellungsname:***Ein eindeutiger Name für die Modellimplementierung*
    - **Bereitstellungstyp**: Globaler Standard
    - **Automatische Versionsaktualisierung**: Aktiviert
    - **Modellversion**: *Wählen Sie die neueste verfügbare Version aus.*
    - **Verbundene AI-Ressource**: *Wählen Sie Ihre Azure OpenAI-Ressourcenverbindung*
    - **Tokens pro Minute Ratenlimit (Tausende)**: 50K *(oder das in Ihrem Abonnement verfügbare Maximum, wenn weniger als 50K)*
    - **Inhaltsfilter**: StandardV2 

    > **Hinweis:** Durch das Verringern des TPM wird die Überlastung des Kontingents vermieden, das in dem von Ihnen verwendeten Abonnement verfügbar ist. 50.000 TPM sollten für die in dieser Übung verwendeten Daten ausreichend sein. Wenn Ihr verfügbares Kontingent darunter liegt, können Sie die Übung zwar abschließen, müssen aber möglicherweise warten und die Prompts erneut senden, wenn das Kontingent überschritten wird.

1. Warten Sie, bis die Bereitstellung abgeschlossen ist.

## Erstellen einer Agent-Client-App

Jetzt können Sie eine Client-App erstellen, die einen Agent und eine benutzerdefinierte Funktion definiert. Etwas Code wurde für Sie in einem GitHub-Repository bereitgestellt.

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

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_connection_string** durch die Verbindungszeichenfolge für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrem gpt-4-Modell-Einsatz zugewiesen haben.
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Schreiben von Code für eine Agent-App

> **Tipp**: Achten Sie beim Hinzufügen von Code auf den richtigen Einzug. Verwenden Sie die vorhandenen Kommentare als Leitfaden, und geben Sie den neuen Code auf derselben Ebene des Einzugs ein.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Agent-Codedatei zu bearbeiten:

    ```
   code semantic-kernel.py
    ```

1. Sehen Sie sich den Code in dieser Datei an. Sie enthält folgende Elemente:
    - Einige **Importanweisungen** zum Hinzufügen von Verweisen auf häufig verwendete Namespaces
    - Eine *Hauptfunktion*, die eine Datei mit Spesendaten lädt, den Benutzer nach Anweisungen fragt und dann aufruft ...
    - Eine **process_expenses_data**-Funktion, in der der Code zum Erstellen und Verwenden des Agents hinzugefügt werden muss
    - Eine **EmailPlugin**-Klasse, die eine Kernelfunktion mit dem Namen **send_email** enthält. Diese wird von Ihrem Agent verwendet, um die Funktionalität zum Senden einer E-Mail zu simulieren.

1. Suchen Sie oben in der Datei nach der vorhandenen **Importanweisung** den Kommentar **Verweise hinzufügen**, und fügen Sie den folgenden Code hinzu, um auf die Namespaces in den Bibliotheken zu verweisen, die Sie zum Implementieren Ihres Agents benötigen:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity.aio import DefaultAzureCredential
   from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AzureAIAgentThread
   from semantic_kernel.functions import kernel_function
   from typing import Annotated
    ```

1. Suchen Sie unten in der Datei den Kommentar **Erstellen eines Plug-Ins für die E-Mail-Funktionalität**, und fügen Sie den folgenden Code hinzu, um eine Klasse für ein Plug-In zu definieren, das eine Funktion enthält, die Ihr Agent zum Senden von E-Mails verwendet (Plug-Ins sind eine Möglichkeit, benutzerdefinierte Funktionen zu Semantischer-Kernel-Agents hinzuzufügen).

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

    > **Hinweis**: Die Funktion *simuliert* das Senden einer E-Mail durch Drucken an die Konsole. In einer echten Anwendung würden Sie einen SMTP-Dienst oder ähnliches verwenden, um die E-Mail tatsächlich zu senden!

1. Sichern Sie oberhalb des neuen Klassencodes **EmailPlugin** in der Funktion **create_expense_claim** den Kommentar **Konfigurationseinstellungen abrufen**, und fügen Sie den folgenden Code hinzu, um die Konfigurationsdatei zu laden und ein Objekt **AzureAIAgentSettings** zu erstellen (das automatisch die Azure KI-Agent-Einstellungen aus der Konfiguration enthält).

    (Achten Sie darauf, die Einzugsebene beizubehalten)

    ```python
   # Get configuration settings
   load_dotenv()
   ai_agent_settings = AzureAIAgentSettings()
    ```

1. Suchen Sie den Kommentar **Verbindung mit dem Azure AI Foundry-Projekt herstellen** und fügen Sie den folgenden Code hinzu, um eine Verbindung mit Ihrem Azure AI Foundry-Projekt herzustellen. Verwenden Sie dazu die Azure-Anmeldeinformationen, mit denen Sie derzeit angemeldet sind:

    (Achten Sie darauf, die Einzugsebene beizubehalten)

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

1. Suchen Sie den Kommentar **Definieren eines Azure KI-Agents, der eine Spesenabrechnungs-E-Mail** sendet, und fügen Sie den folgenden Code hinzu, um eine Azure KI-Agent-Definition für Ihren Agent zu erstellen.

    (Achten Sie darauf, die Einzugsebene beizubehalten)

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

1. Suchen Sie den Kommentar **Erstellen eines Semantischer-Kernel-Agents**, und fügen Sie den folgenden Code hinzu, um ein Semantischer-Kernel-Agent-Objekt für Ihren Azure KI-Agent zu erstellen, und fügen Sie einen Verweis auf das **EmailPlugin**-Plug-In hinzu.

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

1. Geben Sie im Befehlszeilenbereich von Cloud Shell unterhalb des Code-Editors den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
    az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden – obwohl die Cloud Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist bereits die Verwendung von *az login* ausreichend. Wenn Sie jedoch über Abonnements in mehreren Mandanten verfügen, müssen Sie den Mandanten möglicherweise mithilfe des Parameters *-tenant* angeben. Details finden Sie unter [Interaktives Anmelden bei Azure mithilfe der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen zum Öffnen der Anmeldeseite in einer neuen Registerkarte, und geben Sie den bereitgestellten Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry-Hub enthält, wenn Sie dazu aufgefordert werden.
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

In dieser Übung haben Sie das Azure KI Agent-Dienst-SDK und den semantischen Kernel verwendet, um einen Agent zu erstellen.

## Bereinigen

Wenn Sie mit der Erkundung des Azure AI Agent-Dienstes fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
