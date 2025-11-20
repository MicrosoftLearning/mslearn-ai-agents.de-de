---
lab:
  title: Herstellen einer Verbindung mit Remote-Agents mithilfe des A2A-Protokolls
  description: Verwenden Sie das A2A-Protokoll für die Zusammenarbeit mit Remote-Agents.
---

# Herstellen einer Verbindung mit Remote-Agents mithilfe des A2A-Protokolls

In dieser Übung verwenden Sie den Azure KI-Agent-Dienst mit dem A2A-Protokoll, um einfache Remote-Agents zu erstellen, die miteinander interagieren. Diese Agents unterstützen technische Redakteure bei der Vorbereitung ihrer Entwicklerblogbeiträge. Ein Titel-Agent generiert eine Überschrift, und ein Gliederungs-Agent verwendet den Titel, um eine präzise Gliederung für den Artikel zu entwickeln. Erste Schritte

> **Tipp**: Der in dieser Übung verwendete Code basiert auf dem Azure AI Foundry SDK für Python. Sie können ähnliche Lösungen mithilfe der SDKs für Microsoft .NET, JavaScript und Java entwickeln. Ausführliche Informationen finden Sie unter [Azure AI Foundry SDK-Clientbibliotheken](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview).

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Einige der in dieser Übung verwendeten Technologien befinden sich in der Vorschau oder in der aktiven Entwicklung. Es kann zu unerwartetem Verhalten, Warnungen oder Fehlern kommen.

## Erstellen eines Azure KI Foundry-Projekts

Beginnen wir mit dem Erstellen eines Azure AI Foundry-Projekts.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipps oder Schnellstartfenster, die bei der ersten Anmeldung geöffnet werden, und verwenden Sie gegebenenfalls das Logo **Azure AI Foundry** oben links, um zur Startseite zu navigieren, die ähnlich wie die folgende Abbildung aussieht (schließen Sie das **Hilfe**-Fenster, falls es geöffnet ist):

    ![Screenshot des Azure KI Foundry-Portals.](./Media/ai-foundry-home.png)

1. Wählen Sie auf der Startseite **Agent erstellen**.
1. Wenn Sie zur Erstellung eines Projekts aufgefordert werden, geben Sie einen gültigen Namen für Ihr Projekt ein und entfalten Sie die Option **Erweiterte Optionen**.
1. Bestätigen Sie die folgenden Einstellungen für Ihr Projekt:
    - **Azure KI Foundry-Ressource**: *Ein gültiger Name für Ihre Azure KI Foundry-Ressource*
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Region**: Wählen Sie die **empfohlene AI Foundry-Instanz aus***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Klicken Sie auf **Erstellen**, und warten Sie, bis das Projekt erstellt wird.
1. Wenn Sie dazu aufgefordert werden, stellen Sie ein **gpt-4o**-Modell entweder mithilfe der Bereitstellungsoption *Globaler Standard* oder *Standard* bereit (je nach Kontingentverfügbarkeit).

    >**Hinweis:** Wenn ein Kontingent verfügbar ist, kann ein GPT-4o-Basismodell automatisch bereitgestellt werden, wenn Sie Ihren Agent und Ihr Projekt erstellen.

1. Wenn Ihr Projekt erstellt wird, wird der Agents-Playground geöffnet.

1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    ![Screenshot einer Projektübersichtsseite von Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Kopieren Sie die Werte für den **Endpunkt des Azure AI Foundry-Projekts** in einen Notizblock, da Sie diese für die Verbindung mit Ihrem Projekt in einer Clientanwendung benötigen.

## Erstellen einer A2A-Anwendung

Jetzt können Sie eine Client-App erstellen, die einen Agent verwendet. Ein Teil des Codes wurde für Sie in einem GitHub-Repository bereitgestellt.

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
   cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
   ls -a -l
    ```

    Die bereitgestellten Dateien umfassen Folgendes:
    ```output
    python
    ├── outline_agent/
    │   ├── agent.py
    │   ├── agent_executor.py
    │   └── server.py
    ├── routing_agent/
    │   ├── agent.py
    │   └── server.py
    ├── title_agent/
    │   ├── agent.py
    |   ├── agent_executor.py
    │   └── server.py
    ├── client.py
    └── run_all.py
    ```

    Jeder Agent-Ordner enthält den Azure KI-Agent-Code und einen Server zum Hosten des Agents. Der **Routing-Agent** ist für die Ermittlung und Kommunikation mit den **Titel**- und **Gliederungs**-Agents verantwortlich. Mit dem **Client** können Benutzer Prompts an den Routing-Agent senden. `run_all.py` startet alle Server und führt den Client aus.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents a2a-sdk
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert aus der Projektseite **Übersicht** im Azure AI Foundry-Portal), und stellen Sie sicher, dass die Variable MODEL_DEPLOYMENT_NAME auf den Namen Ihrer Modellbereitstellung festgelegt ist (dies sollte *gpt-4o* sein).
1. Nachdem Sie den Platzhalter ersetzt haben, speichern Sie Ihre Änderungen mit dem Befehl **STRG+S** und schließen Sie den Code-Editor mit dem Befehl **STRG+Q**, wobei die Cloud Shell-Befehlszeile geöffnet bleibt.

### Erstellen eines auffindbaren Agents

In dieser Aufgabe erstellen Sie den Titel-Agent, der Redakteuren hilft, ansprechende Überschriften für ihre Artikel zu finden. Sie definieren auch die Skills und die Karte des Agents, die vom A2A-Protokoll benötigt werden, um den Agent auffindbar zu machen.

1. Navigieren Sie zum Verzeichnis `title_agent`:

    ```
   cd title_agent
    ```

> **Tipp**: Achten Sie beim Hinzufügen von Code auf den richtigen Einzug. Verwenden Sie die Einzugsebenen für Kommentare als Orientierungshilfe.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Codedatei zu bearbeiten:

    ```
   code agent.py
    ```

1. Suchen Sie den Kommentar **Create the agents client**, und fügen Sie den folgenden Code hinzu, um eine Verbindung zum Azure KI-Projekt herzustellen:

    > **Tipp**: Achten Sie darauf, die richtige Einzugsebene beizubehalten.

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. Suchen Sie den Kommentar **Create the title agent**, und fügen Sie den folgenden Code hinzu, um den Agent zu erstellen:

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. Suchen Sie den Kommentar **Create a thread for the chat session**, und fügen Sie den folgenden Code hinzu, um den Chatthread zu erstellen:

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. Suchen Sie den Kommentar **Send user message**, und fügen Sie diesen Code hinzu, um den Prompt des Benutzers zu übermitteln:

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. Fügen Sie unter dem Kommentar **Create and run the agent** den folgenden Code hinzu, um die Antwortgenerierung des Agents zu initiieren:

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    Der im Rest der Datei bereitgestellte Code verarbeitet die Antwort des Agents und gibt sie zurück. 

1. Speichern Sie die Codedatei (*STRG+S*). Jetzt können Sie die Skills und die Karte des Agents mit dem A2A-Protokoll teilen. 

1. Geben Sie den folgenden Befehl ein, um die `server.py`-Datei des Titel-Agents zu bearbeiten.  

    ```
   code server.py
    ```

1. Suchen Sie nach dem Kommentar **Define agent skills**, und fügen Sie den folgenden Code hinzu, um die Funktion des Agents anzugeben:

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. Suchen Sie den Kommentar **Create agent card**, und fügen Sie diesen Code hinzu, um die Metadaten zu definieren, die den Agent auffindbar machen:

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='AI Foundry Title Agent',
       description='An intelligent title generator agent powered by Azure AI Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. Suchen Sie den Kommentar **Create agent executor**, und fügen Sie den folgenden Code hinzu, um den Agent-Executor mithilfe der Agent-Karte zu initialisieren:

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    Der Agent-Executor fungiert als Wrapper für den Titel-Agent, den Sie erstellt haben.

1. Suchen Sie den Kommentar **Create request handler**, und fügen Sie Folgendes hinzu, um eingehende Anforderungen mit dem Executor zu verarbeiten:

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. Fügen Sie unter dem Kommentar **Create A2A application** den folgenden Code hinzu, um die A2A-kompatible Anwendungsinstanz zu erstellen:

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```
    
    Dieser Code erstellt einen A2A-Server, der die Informationen des Titel-Agents weitergibt und eingehende Anforderungen für diesen Agent mithilfe des Titel-Agent-Executors verarbeitet.

1. Speichern Sie die Codedatei (*STRG+S*), wenn Sie fertig sind.

### Aktivieren von Nachrichten zwischen den Agents

In dieser Aufgabe verwenden Sie das A2A-Protokoll, um dem Routing-Agent das Senden von Nachrichten an die anderen Agents zu ermöglichen. Sie lassen auch zu, dass der Titel-Agent Nachrichten empfängt, indem Sie die Executor-Klasse des Agents implementieren.

1. Navigieren Sie zum Verzeichnis `routing_agent`:

    ```
   cd ../routing_agent
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Codedatei zu bearbeiten:

    ```
   code agent.py
    ```

    Der Routing-Agent fungiert als Orchestrator, der Benutzernachrichten verarbeitet und bestimmt, welcher Remote-Agent die Anforderung verarbeiten soll.

    Wenn eine Benutzernachricht empfangen wird, macht der Routing-Agent Folgendes:
    - Er startet einen Konversationsthread.
    - Er verwendet die `create_and_process`-Methode, um den am besten geeigneten Agent für die Nachricht des Benutzers zu ermitteln.
    - Die Nachricht wird mithilfe der `send_message`-Funktion über HTTP an den entsprechenden Agent weitergeleitet.
    - Der Remote-Agent verarbeitet die Nachricht und gibt eine Antwort zurück.

    Der Routing-Agent erfasst schließlich die Antwort und gibt sie über den Thread an den Benutzer zurück.

    Beachten Sie, dass die `send_message`-Methode asynchron ist und auf diese gewartet werden muss, damit der Agent erfolgreich ausgeführt wird.

1. Fügen Sie den folgenden Code unter dem Kommentar **Retrieve the remote agent's A2A client using the agent name** hinzu:

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. Suchen Sie den Kommentar **Construct the payload to send to the remote agent**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. Suchen Sie den Kommentar **Wrap the payload in a SendMessageRequest object**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. Fügen Sie den folgenden Code unter dem Kommentar **Send the message to the remote agent client and await the response** hinzu:

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```


1. Speichern Sie die Codedatei (*STRG+S*), wenn Sie fertig sind. Jetzt kann der Routing-Agent Nachrichten ermitteln und an den Titel-Agent senden. Erstellen wir nun den Executor-Code des Agents, um diese eingehenden Nachrichten vom Routing-Agent zu verarbeiten.

1. Navigieren Sie zum Verzeichnis `title_agent`:

    ```
   cd ../title_agent
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Codedatei zu bearbeiten:

    ```
   code agent_executor.py
    ```

    Die `AgentExecutor`-Klassenimplementierung muss die Methoden `execute` und `cancel` enthalten. Die cancel-Methode wurde für Sie bereitgestellt. Die `execute`-Methode enthält ein `TaskUpdater`-Objekt, das Ereignisse verwaltet und dem Aufrufer signalisiert, wenn der Task abgeschlossen ist. Fügen wir nun die Logik für die Taskausführung hinzu.

1. Fügen Sie in der `execute`-Methode den folgenden Code unter dem Kommentar **Process the request** hinzu:

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. Fügen Sie in der `_process_request`-Methode den folgenden Code unter dem Kommentar **Get the title agent** hinzu:

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. Fügen Sie den folgenden Code unter dem Kommentar **Update the task status** hinzu:

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. Suchen Sie den Kommentar, **Run the agent conversation** aus, und fügen Sie den folgenden Code hinzu:

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. Suchen Sie den Kommentar **Update the task with the responses**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. Suchen Sie den Kommentar **Mark the task as complete**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    Ihr Titel-Agent wurde nun mit einem Agent-Executor umschlossen, den das A2A-Protokoll zum Verarbeiten von Nachrichten verwendet. Gut gemacht!

### Bei Azure anmelden und die App ausführen

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
    az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry Hub enthält, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
    cd ..
    python run_all.py
    ```
    
    Die Anwendung wird mit den Anmeldeinformationen für Ihre authentifizierte Azure-Sitzung ausgeführt, um eine Verbindung zu Ihrem Projekt herzustellen und den Agent zu erstellen und auszuführen. Beim Start sollte auf jedem Server eine Ausgabe angezeigt werden.

1. Warten Sie, bis die Eingabeaufforderung angezeigt wird, und geben Sie dann einen Prompt ein, z. B.:

    ```
   Create a title and outline for an article about React programming.
    ```

    Nach einigen Augenblicken sollte eine Antwort vom Agent mit den Ergebnissen angezeigt werden.

1. Geben Sie `quit` ein, um das Programm und die Server zu beenden.
    
## Zusammenfassung

In dieser Übung haben Sie das SDK des Azure KI-Agent-Diensts und das A2A Python-SDK verwendet, um eine Remotelösung mit mehreren Agents zu erstellen. Sie haben einen auffindbaren, A2A-kompatiblen Agent erstellt und einen Routing-Agent für den Zugriff auf die Skills des Agents eingerichtet. Sie haben auch einen Agent-Executor implementiert, um eingehende A2A-Nachrichten zu verarbeiten und Aufgaben zu verwalten. Gut gemacht!

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
