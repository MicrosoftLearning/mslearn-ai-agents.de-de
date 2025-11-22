---
lab:
  title: Verwenden einer benutzerdefinierten Funktion in einem KI-Agent
  description: 'Erfahren Sie, wie Sie Funktionen verwenden, um Ihren Agents benutzerdefinierte Funktionen hinzuzufügen.'
---

# Verwenden einer benutzerdefinierten Funktion in einem KI-Agent

In dieser Übung erfahren Sie, wie Sie einen Agent erstellen, der benutzerdefinierte Funktionen als Tool zum Ausführen von Aufgaben verwenden kann. Sie erstellen einen einfachen technischen Support Agent, der Details zu einem technischen Problem sammeln und ein Supportticket generieren kann.

> **Tipp**: Der in dieser Übung verwendete Code basiert auf dem Microsoft Foundry SDK für Python. Sie können ähnliche Lösungen mithilfe der SDKs für Microsoft .NET, JavaScript und Java entwickeln. Ausführliche Informationen finden Sie unter [Microsoft Foundry SDK-Clientbibliotheken](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview).

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Einige der in dieser Übung verwendeten Technologien befinden sich in der Vorschau oder in der aktiven Entwicklung. Es kann zu unerwartetem Verhalten, Warnungen oder Fehlern kommen.

## Erstellen eines Foundry-Projekts

Beginnen wir mit dem Erstellen eines Foundry-Projekts.

1. Öffnen Sie in einem Webbrowser das [Foundry-Portal](https://ai.azure.com) unter `https://ai.azure.com` und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipp- oder Schnellstartbereiche, die beim ersten Anmelden geöffnet werden, und verwenden Sie bei Bedarf zum Navigieren zur Startseite das **Foundry**-Logo oben links, das ähnlich wie die folgende Abbildung aussieht (schließen Sie den Bereich **Hilfe**, wenn er geöffnet ist):

    ![Screenshot des Foundry-Portals.](./Media/ai-foundry-home.png)

    > **Wichtig:** Stellen Sie sicher, dass der Umschalter **Neues Foundry** für diese Übung *deaktiviert* ist.

1. Wählen Sie auf der Startseite **Agent erstellen**.
1. Wenn Sie zur Erstellung eines Projekts aufgefordert werden, geben Sie einen gültigen Namen für Ihr Projekt ein und entfalten Sie die Option **Erweiterte Optionen**.
1. Bestätigen Sie die folgenden Einstellungen für Ihr Projekt:
    - **Foundry-Ressource**: *Ein gültiger Name für Ihre Foundry-Ressource*
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Region**: Wählen Sie die **empfohlene AI Foundry-Instanz aus***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Klicken Sie auf **Erstellen**, und warten Sie, bis das Projekt erstellt wird.
1. Wenn Sie dazu aufgefordert werden, stellen Sie ein **gpt-4o**-Modell entweder mithilfe der Bereitstellungsoption *Globaler Standard* oder *Standard* bereit (je nach Kontingentverfügbarkeit).

    >**Hinweis:** Wenn ein Kontingent verfügbar ist, kann ein GPT-4o-Basismodell automatisch bereitgestellt werden, wenn Sie Ihren Agent und Ihr Projekt erstellen.

1. Wenn Ihr Projekt erstellt wird, wird der Agents-Playground geöffnet.

1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    ![Screenshot der Übersicht eines Foundry-Projekts.](./Media/ai-foundry-project.png)

1. Kopieren Sie die Werte des **Foundry-Projektendpunkts** in einen Editor, da Sie diese für die Verbindung mit Ihrem Projekt in einer Clientanwendung benötigen.

## Entwickeln eines Agents, der Funktionstools verwendet

Nachdem Sie nun Ihr Projekt in AI Foundry erstellt haben, entwickeln wir eine App, die einen Agent mit benutzerdefinierten Funktionstools implementiert.

### Klonen des Repositorys mit dem Anwendungscode

1. Öffnen Sie eine neue Browserregisterkarte (wobei das Foundry-Portal auf der vorhandenen Registerkarte geöffnet bleibt). Wechseln Sie dann in der neuen Registerkarte zum [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldedaten an, wenn Sie dazu aufgefordert werden.

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

    Die bereitgestellten Dateien enthalten Anwendungscode und eine Datei für Einstellungen der Konfiguration.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents
    ```

    >**Hinweis:** Sie können alle während der Installation der Bibliothek angezeigten Warn- oder Fehlermeldungen ignorieren.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert aus der Projektseite **Übersicht** im Foundry-Portal) und stellen Sie sicher, dass die Variable MODEL_DEPLOYMENT_NAME auf den Namen Ihrer Modellimplementierung festgelegt ist (dies sollte *gpt-4o* sein).
1. Nachdem Sie den Platzhalter ersetzt haben, speichern Sie Ihre Änderungen mit dem Befehl **STRG+S** und schließen Sie den Code-Editor mit dem Befehl **STRG+Q**, wobei die Cloud Shell-Befehlszeile geöffnet bleibt.

### Definieren einer benutzerdefinierten Funktion

1. Geben Sie den folgenden Befehl ein, um die Codedatei zu bearbeiten, die für Ihren Funktionscode bereitgestellt wurde:

    ```
   code user_functions.py
    ```

1. Suchen Sie den Kommentar **Erstellen Sie eine Funktion zum Einreichen eines Supporttickets** und fügen Sie den folgenden Code ein, der eine Ticketnummer generiert und ein Supportticket als Textdatei speichert.

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

1. Suchen Sie den Kommentar **Definieren Sie eine Reihe von aufrufbaren Funktionen** und fügen Sie den folgenden Code hinzu, der statisch eine Reihe von aufrufbaren Funktionen in dieser Codedatei definiert (in diesem Fall gibt es nur eine - aber in einer echten Lösung haben Sie vielleicht mehrere Funktionen, die Ihr Agent aufrufen kann):

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. Speichern Sie die Datei (*CTRL+S*).

### Schreiben von Code zum Implementieren eines Agents, der Ihre Funktion verwenden kann

1. Geben Sie den folgenden Befehl ein, um mit der Bearbeitung des Agent-Codes zu beginnen.

    ```
    code agent.py
    ```

    > **Tipp**: Achten Sie beim Hinzufügen von Code zur Codedatei darauf, den richtigen Einzug beizubehalten.

1. Überprüfen Sie den vorhandenen Code, der die Anwendungskonfigurations-Einstellungen abruft, und richtet eine Schleife ein, in der die benutzende Person Eingabeaufforderungen für den Agent eingeben kann. Der Rest der Datei enthält Kommentare, in die Sie den notwendigen Code für die Implementierung Ihres technischen Support-Agent einfügen.
1. Suchen Sie den Kommentar **Referenzen hinzufügen** und fügen Sie den folgenden Code hinzu, um die Klassen zu importieren, die Sie benötigen, um einen Azure AI-Agent zu erstellen, der Ihren Funktionscode als Tool verwendet:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, ToolSet, ListSortOrder, MessageRole
   from user_functions import user_functions
    ```

1. Suchen Sie den Kommentar **Mit dem Agent-Client verbinden** und fügen Sie den folgenden Code hinzu, um mit den aktuellen Azure-Anmeldeinformationen eine Verbindung zum Azure KI-Projekt herzustellen.

    > **Tipp**: Achten Sie darauf, die richtige Einzugsebene beizubehalten.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
    ```
    
1. Suchen Sie den Abschnitt **Definieren Sie einen Agenten, der die benutzerdefinierten Funktionen verwenden kann**, und fügen Sie den folgenden Code hinzu, um Ihren Funktionscode zu einem Toolset hinzuzufügen, und erstellen Sie dann einen Agenten, der das Toolset und einen Thread verwenden kann, auf dem die Chatsitzung läuft.

    ```python
   # Define an agent that can use the custom functions
   with agent_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        agent_client.enable_auto_function_calls(toolset)
            
        agent = agent_client.create_agent(
            model=model_deployment,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = agent_client.threads.create()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. Suchen Sie den Kommentar **Senden Sie eine Eingabeaufforderung an den Agent** und fügen Sie den folgenden Code hinzu, um die Eingabeaufforderung der Benutzenden als Nachricht hinzuzufügen und den Thread auszuführen.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```

    > **Hinweis**: Durch die Verwendung der Methode **create_and_process** zum Ausführen des Threads kann der Agent Ihre Funktionen automatisch finden und anhand ihrer Namen und Parameter auswählen. Alternativ könnten Sie die Methode **create_run** verwenden. In diesem Fall wären Sie dafür verantwortlich, Code zu schreiben, der den Ausführungsstatus abfragt, um festzustellen, wann ein Funktionsaufruf erforderlich ist, die Funktion aufruft und die Ergebnisse an den Agent zurückgibt.

1. Suchen Sie den Kommentar **Prüfen Sie den Ausführungsstatus auf Fehler** und fügen Sie den folgenden Code ein, um alle auftretenden Fehler anzuzeigen.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Suchen Sie den Kommentar **Die letzte Antwort des Agents anzeigen** und fügen Sie den folgenden Code hinzu, um die Nachrichten aus dem abgeschlossenen Thread abzurufen und die letzte vom Agent gesendete Nachricht anzuzeigen.

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Suchen Sie den Kommentar **Abrufen von aufgezeichneten Unterhaltungen** und fügen Sie den folgenden Code hinzu, um die Nachrichten aus dem Konversations-Thread in chronologischer Reihenfolge auszudrucken:

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. Suchen Sie den Kommentar **Aufräumen** und fügen Sie den folgenden Code ein, um den Agent und den Thread zu löschen, wenn er nicht mehr benötigt wird.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
   print("Deleted agent")
    ```

1. Überprüfen Sie den Code mithilfe der Kommentare, um zu verstehen, wie er:
    - Fügt Ihren Satz von benutzerdefinierten Funktionen zu einem Toolset hinzu.
    - Erstellt einen Agent, der das Toolset verwendet.
    - Führt einen Thread mit einer Eingabeaufforderungsmeldung der benutzenden Person aus.
    - Überprüft den Status der Ausführung, falls ein Fehler auftritt
    - Ruft die Nachrichten aus dem abgeschlossenen Thread ab und zeigt die letzte vom Agenten gesendete Nachricht an.
    - Zeigt die aufgezeichneten Unterhaltungen an.
    - Löscht den Agent und den Thread, wenn er nicht mehr benötigt wird.

1. Speichern Sie die Codedatei (*STRG+S*), wenn Sie fertig sind. Sie können auch den Code-Editor (*STRG+Q*) schließen. Möglicherweise möchten Sie ihn jedoch geöffnet lassen, falls Sie Änderungen an dem Code vornehmen müssen, den Sie hinzugefügt haben. Lassen Sie in beiden Fällen den Befehlszeilenbereich der Cloud Shell geöffnet.

### Bei Azure anmelden und die App ausführen

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
    az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab und wählen Sie das Abonnement mit Ihrem Foundry-Hub aus, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
   python agent.py
    ```
    
    Die Anwendung wird mit den Anmeldeinformationen für Ihre authentifizierte Azure-Sitzung ausgeführt, um eine Verbindung zu Ihrem Projekt herzustellen und den Agent zu erstellen und auszuführen.

1. Wenn Sie dazu aufgefordert werden, geben Sie eine Eingabeaufforderung, wie etwa:

    ```
   I have a technical problem
    ```

    > **Tipp**: Wenn die App fehlschlägt, weil das Ratenlimit überschritten wird. Warten Sie einige Sekunden, und versuchen Sie es noch mal. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

1. Zeigen Sie die Antwort an. Der Agent kann Sie um Ihre E-Mail-Adresse und eine Beschreibung des Problems bitten. Sie können eine beliebige E-Mail-Adresse (zum Beispiel `alex@contoso.com`) und eine beliebige Problembeschreibung (zum Beispiel `my computer won't start`) verwenden.

    Wenn es über genügend Informationen verfügt, sollte der Agent ihre Funktion nach Bedarf verwenden.

1. Sie können die Unterhaltung fortsetzen, wenn Sie möchten. Der Thread ist *statusbehaftet*,er behält die aufgezeichneten Unterhaltungen bei, was bedeutet, dass der Agent den vollständigen Kontext für jede Antwort hat. Geben Sie `quit` ein, wenn Sie fertig sind.
1. Überprüfen Sie die Unterhaltungsnachrichten, die aus dem Thread abgerufen wurden, und die Tickets, die generiert wurden.
1. Das Tool sollte Supporttickets im App-Ordner gespeichert haben. Sie können den Befehl `ls` verwenden, um zu prüfen, und dann den Befehl `cat` verwenden, um den Inhalt der Datei anzuzeigen, etwa so:

    ```
   cat ticket-<ticket_num>.txt
    ```

## Bereinigen

Nachdem Sie die Übung beendet haben, sollten Sie die von Ihnen erstellten Cloud-Ressourcen löschen, um eine unnötige Ressourcennutzung zu vermeiden.

1. Öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Hub-Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
