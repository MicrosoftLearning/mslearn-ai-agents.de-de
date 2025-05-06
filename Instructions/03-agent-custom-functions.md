---
lab:
  title: Verwenden einer benutzerdefinierten Funktion in einem KI-Agent
  description: 'Erfahren Sie, wie Sie Funktionen verwenden, um Ihren Agents benutzerdefinierte Funktionen hinzuzufügen.'
---

# Verwenden einer benutzerdefinierten Funktion in einem KI-Agent

In dieser Übung erfahren Sie, wie Sie einen Agent erstellen, der benutzerdefinierte Funktionen als Tool zum Ausführen von Aufgaben verwenden kann.

Sie erstellen einen einfachen Agent für technischen Support, der Details zu einem technischen Problem sammeln und ein Supportticket generieren kann.

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

## Entwickeln eines Agents, der Funktionstools verwendet

Nachdem Sie nun Ihr Projekt in AI Foundry erstellt haben, entwickeln wir eine App, die einen Agent mit benutzerdefinierten Funktionstools implementiert.

### Klonen des Repositorys mit dem Anwendungscode

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

1. Geben Sie den folgenden Befehl ein, um das Arbeitsverzeichnis in den Ordner mit den Codedateien zu ändern und alle aufzulisten.

    ```
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    Die bereitgestellten Dateien enthalten Anwendungscode und eine Datei für Konfigurationseinstellungen.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**Hinweis:** Sie können Warnungs- oder Fehlermeldungen ignorieren, die während der Installation der Bibliothek angezeigt werden.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_connection_string** durch die Verbindungszeichenfolge für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrem gpt-4-Modell-Einsatz zugewiesen haben.
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Definieren einer benutzerdefinierten Funktion

1. Geben Sie den folgenden Befehl ein, um die für Ihren Funktionscode bereitgestellte Codedatei zu bearbeiten:

    ```
   code user_functions.py
    ```

1. Suchen Sie den Kommentar **Eine Funktion erstellen, um ein Supportticket zu übermitteln**, und fügen Sie den folgenden Code hinzu, der eine Ticketnummer generiert und ein Supportticket als Textdatei speichert.

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. Suchen Sie den Kommentar **Definieren eines Satzes aufrufbarer Funktionen**, und fügen Sie den folgenden Code hinzu, der in dieser Codedatei einen Satz aufrufbarer Funktionen definiert (in diesem Fall gibt es nur eine – aber in einer echten Lösung haben Sie möglicherweise mehrere Funktionen, die Ihr Agent aufrufen kann):

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Speichern Sie die Datei (*STRG+S*).

### Schreiben von Code zum Implementieren eines Agents, der Ihre Funktion verwenden kann

1. Geben Sie den folgenden Befehl ein, um den Code des Agents zu bearbeiten.

    ```
    code agent.py
    ```

    > **Tipp**: Achten Sie beim Hinzufügen von Code zur Codedatei auf den korrekten Einzug.

1. Überprüfen Sie den vorhandenen Code, der die Anwendungskonfigurationseinstellungen abruft und eine Schleife einrichtet, in der Benutzende Prompts für den Agent eingeben können. Der Rest der Datei enthält Kommentare, in denen Sie dann den erforderlichen Code hinzufügen, um Ihren Agent für technischen Support zu implementieren.
1. Suchen Sie den Kommentar **Verweise hinzufügen**, und fügen Sie den folgenden Code hinzu, um die Klassen zu importieren, die Sie zum Erstellen eines Azure KI-Agents benötigen, der Ihren Funktionscode als Tool verwendet:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. Finden Sie den Kommentar **Verbindung mit dem Azure AI Foundry-Projekt herstellen** und fügen Sie den folgenden Code hinzu, um sich mit Ihrem Azure KI-Projekt zu verbinden. Verwenden Sie dazu die Azure-Anmeldeinformationen, mit denen Sie derzeit angemeldet sind.

    > **Tipp**: Achten Sie darauf, die richtige Einzugsebene beizubehalten.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. Suchen Sie den Kommentar **Definieren eines Agents, der den Abschnitt „Benutzerdefinierte Funktionen“ verwenden kann**, und fügen Sie den folgen Code hinzu, um Ihren Funktionscode einem Toolset hinzuzufügen. Erstellen Sie dann einen Agent, der das Toolset und einen Thread zum Ausführen der Chatsitzung verwenden kann.

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        project_client.agents.enable_auto_function_calls(toolset=toolset)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Suchen Sie den Kommentar **Senden eines Prompts an den Agent**, und fügen Sie den folgenden Code hinzu, um den Prompt von Benutzenden als Nachricht hinzuzufügen und den Thread auszuführen.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Hinweis**: Die Verwendung der Methode **create_and_process_run** zum Ausführen des Threads ermöglicht es dem Agent, Ihre Funktionen automatisch zu finden und sie basierend auf ihren Namen und Parametern zu verwenden. Alternativ können Sie die Methode **create_run** verwenden. In diesem Fall wären Sie für das Schreiben von Code zur Abfrage des Ausführungsstatus verantwortlich, um festzustellen, wann ein Funktionsaufruf erforderlich ist, die Funktion aufzurufen und die Ergebnisse an den Agent zurückzugeben.

1. Suchen Sie den Kommentar **Überprüfen Sie den Ausführungsstatus auf Fehler**, und fügen Sie den folgenden Code hinzu, um alle aufgetretenen Fehler anzuzeigen.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Suchen Sie den Kommentar **Die neueste Antwort des Agents anzeigen**, und fügen Sie den folgenden Code hinzu, um die Nachrichten aus dem abgeschlossenen Thread abzurufen und die letzte anzuzeigen, die vom Agent gesendet wurde.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Suchen Sie den Kommentar **Abrufen von aufgezeichneten Unterhaltungen**, der nach Dem Ende der Schleife liegt, und fügen Sie den folgenden Code hinzu, um die Nachrichten aus dem Unterhaltungsthread auszudrucken; die Reihenfolge wird umkehrt, um sie in chronologischer Reihenfolge anzuzeigen.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. Suchen Sie den Kommentar **Aufräumen**, und fügen Sie den folgenden Code hinzu, um den Agent und den Thread zu löschen, wenn sie nicht mehr benötigt werden.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. Überprüfen Sie den Code mithilfe der Kommentare, um zu verstehen, wie er:
    - Ihren Satz von benutzerdefinierten Funktionen zu einem Toolset hinzufügt
    - Einen Agent erstellt, der das Toolset verwendet.
    - Führt einen Thread mit einer Prompt-Nachricht der Benutzenden aus.
    - Überprüft den Status der Ausführung, falls ein Fehler auftritt.
    - Ruft die Nachrichten aus dem abgeschlossenen Thread ab und zeigt die zuletzt vom Agent gesendete an.
    - Zeigt die aufgezeichneten Unterhaltungen an.
    - Löscht den Agent und den Thread, wenn sie nicht mehr benötigt werden.

1. Speichern Sie die Codedatei (*STRG+S*), wenn Sie fertig sind. Sie können auch den Code-Editor (*STRG+Q*) schließen. Möglicherweise möchten Sie ihn jedoch geöffnet lassen, falls Sie Änderungen an dem Code vornehmen müssen, den Sie hinzugefügt haben. Lassen Sie in beiden Fällen den Befehlszeilenbereich von Cloud Shell geöffnet.

### Bei Azure anmelden und die App ausführen

1. Geben Sie im Befehlszeilenbereich von Cloud Shell den folgenden Befehl ein, um sich bei der App anzumelden:

    ```
    az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden – obwohl die Cloud Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist bereits die Verwendung von *az login* ausreichend. Wenn Sie jedoch über Abonnements in mehreren Mandanten verfügen, müssen Sie den Mandanten möglicherweise mithilfe des Parameters *-tenant* angeben. Details finden Sie unter [Interaktives Anmelden bei Azure mithilfe der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen zum Öffnen der Anmeldeseite in einer neuen Registerkarte, und geben Sie den bereitgestellten Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry-Hub enthält, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
   python agent.py
    ```
    
    Die Anwendung wird mit den Anmeldeinformationen für Ihre authentifizierte Azure-Sitzung ausgeführt, um eine Verbindung mit Ihrem Projekt herzustellen und den Agent zu erstellen und auszuführen.

1. Wenn Sie dazu aufgefordert werden, geben Sie einen Prompt ein, z. B.:

    ```
   I have a technical problem
    ```

    > **Tipp**: Wenn die App fehlschlägt, weil das Ratenlimit überschritten wird. Warten Sie einige Sekunden, und versuchen Sie es noch mal. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

1. Zeigen Sie die Antwort an. Der Agent kann Ihre E-Mail-Adresse und eine Beschreibung des Problems anfordern. Sie können eine beliebige E-Mail-Adresse (z. B. `alex@contoso.com`) und eine beliebige Problembeschreibung (z. B `my computer won't start`) verwenden.

    Wenn er über genügend Informationen verfügt, sollte der Agent ihre Funktion nach Bedarf verwenden.

1. Sie können die Unterhaltung fortsetzen, wenn Sie möchten. Der Thread ist *zustandsbehaftet*, sodass er die aufgezeichneten Unterhaltungen beibehält, d. h., der Agent hat den vollständigen Kontext für jede Antwort. Wenn Sie fertig sind, geben Sie `quit` ein.
1. Lesen Sie die Unterhaltungsnachrichten, die aus dem Thread abgerufen wurden, und die Tickets, die generiert wurden.
1. Das Tool sollte Supporttickets im App-Ordner gespeichert haben. Sie können den Befehl `ls` zum Überprüfen verwenden und dann den Befehl `cat` verwenden, um die Dateiinhalte wie folgt anzuzeigen:

    ```
   cat ticket-<ticket_num>.txt
    ```

## Bereinigen

Nachdem Sie die Übung beendet haben, sollten Sie die von Ihnen erstellten Cloud-Ressourcen löschen, um eine unnötige Ressourcennutzung zu vermeiden.

1. Öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com`, und zeigen Sie die Inhalte der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.