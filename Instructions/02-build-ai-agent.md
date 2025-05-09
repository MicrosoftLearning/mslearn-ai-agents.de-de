---
lab:
  title: Entwickeln eines KI-Agents
  description: 'Verwenden Sie den Azure AI Agent-Dienst, um einen Agent zu entwickeln, der integrierte Tools verwendet.'
---

# Entwickeln eines KI-Agents

In dieser Übung verwenden Sie Azure AI Agent-Dienst, um einen einfachen Agent zu erstellen, der Daten analysiert und Diagramme erstellt. Der Agent nutzt das eingebaute Tool *Code-Interpreter*, um den Code, der zum Erstellen von Diagrammen als Bilder gebraucht wird, dynamisch zu generieren, und speichert dann die resultierenden Diagramm-Bilder.

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
    - **Standort:** Wählen Sie eine der folgenden Regionen aus:\*
        - eastus
        - eastus2
        - swedencentral
        - westus
        - westus3
    - A**zure KI Services oder Azure OpenAI verbinden**: *Wählen Sie Neuen KI-Dienst erstellen aus*
    - **Azure KI-Suche verbinden**: Verbindung überspringen

    > \* Zum Zeitpunkt des Schreibens unterstützen diese Regionen das gpt-4o-Modell für die Nutzung in Agents. Die Modellverfügbarkeit wird durch regionale Kontingente eingeschränkt. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze erreicht werden, besteht die Möglichkeit, dass Sie ein weiteres Projekt in einer anderen Region erstellen müssen.

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
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    Die bereitgestellten Dateien umfassen Anwendungscode, Konfigurationseinstellungen und Daten.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_connection_string** mit der Verbindungszeichenfolge für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** mit dem Namen, den Sie Ihrer gpt-4o-Modell-Bereitstellung zugewiesen haben.
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Schreiben von Code für eine Agent-App.

> **Tipp**: Achten Sie beim Hinzufügen von Code auf den richtigen Einzug. Verwenden Sie die Einzugsebenen für Kommentare als Orientierungshilfe.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Codedatei zu bearbeiten:

    ```
   code agent.py
    ```

1. Überprüfen Sie den vorhandenen Code, der die Anwendungskonfigurationseinstellungen abruft und Daten aus *data.txt* zur Analyse lädt. Der Rest der Datei enthält Kommentare, in denen Sie den erforderlichen Code hinzufügen, um Ihren Datenanalyse-Agent zu implementieren.
1. Suchen Sie den Kommentar **Add references** und fügen Sie den folgenden Code hinzu, um die Klassen zu importieren, die Sie zum Erstellen eines Azure AI-Agents benötigen, der das integrierte Code-Interpreter-Tool verwendet:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FilePurpose, CodeInterpreterTool
    ```

1. Suchen Sie den Kommentar **Connect to the Azure AI Foundry project** und fügen Sie den folgenden Code hinzu, um eine Verbindung zum Azure KI-Projekt herzustellen.

    > **Tipp**: Achten Sie darauf, die richtige Einzugsebene beizubehalten.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
   with project_client:
    ```

    Der Code stellt mithilfe der aktuellen Azure-Anmeldeinformationen eine Verbindung mit dem Azure AI Foundry-Projekt her. Die abschließende Anweisung *with project_client* startet einen Codeblock, der den Geltungsbereich des Clients definiert und sicherstellt, dass dieser bereinigt wird, wenn der Code innerhalb des Blocks beendet ist.

1. Suchen Sie den Kommentar **Upload the data file and create a CodeInterpreterTool** innerhalb des Blocks *with project_client* und fügen Sie den folgenden Code hinzu, um die Datendatei in das Projekt hochzuladen und ein CodeInterpreterTool zu erstellen, das auf die darin enthaltenen Daten zugreifen kann:

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = project_client.agents.upload_file_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. Suchen Sie den Kommentar **Define an agent that uses the CodeInterpreterTool** und fügen Sie den folgenden Code hinzu, um einen KI-Agent zu definieren, der Daten analysiert und das zuvor definierte Code-Interpreter-Tool verwenden kann:

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = project_client.agents.create_agent(
        model=MODEL_DEPLOYMENT,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. If the user requests a chart, create it and save it as a .png file.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. Suchen Sie den Kommentar **Create a thread for the conversation** und fügen Sie den folgenden Code hinzu, um einen Thread zu starten, in dem die Chat-Sitzung mit dem Agent ausgeführt wird:

    ```python
   # Create a thread for the conversation
   thread = project_client.agents.create_thread()
    ```
    
1. Beachten Sie, dass im nächsten Abschnitt des Codes eine Schleife eingerichtet wird, in der die benutzende Person einen Prompt eingibt, der beendet wird, wenn die Person „quit“ eingibt.

1. Suchen Sie den Kommentar **Send a prompt to the agent** und fügen Sie den folgenden Code hinzu, um eine Benutzermeldung zum Prompt hinzuzufügen (zusammen mit den Daten aus der zuvor geladenen Datei), und führen Sie dann den Thread mit dem Agent aus.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

    run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
     ```

1. Suchen Sie den Kommentar **Prüfen Sie den Ausführungsstatus auf Fehler** und fügen Sie den folgenden Code ein, um alle auftretenden Fehler anzuzeigen.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. Suchen Sie den Kommentar **Die letzte Antwort des Agents anzeigen** und fügen Sie den folgenden Code hinzu, um die Nachrichten aus dem abgeschlossenen Thread abzurufen und die letzte vom Agent gesendete Nachricht anzuzeigen.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. Suchen Sie den Kommentar **Get the conversation history**, der sich nach dem Ende der Schleife befindet, und fügen Sie den folgenden Code ein, um die Nachrichten aus dem Unterhaltungsthread auszudrucken; kehren Sie die Reihenfolge um, um sie in chronologischer Reihenfolge anzuzeigen

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. Suchen Sie den Kommentar **Abrufen aller erzeugten Dateien** und fügen Sie den folgenden Code hinzu, um alle Dateipfad-Anmerkungen aus den Nachrichten zu erhalten (die anzeigen, dass der Agent eine Datei in seinem internen Speicher gespeichert hat) und die Dateien in den Anwendungsordner zu kopieren.

    ```python
   # Get any generated files
   for file_path_annotation in messages.file_path_annotations:
        project_client.agents.save_file(file_id=file_path_annotation.file_path.file_id, file_name=Path(file_path_annotation.text).name)
        print(f"File saved as {Path(file_path_annotation.text).name}")
    ```

1. Suchen Sie den Kommentar **Aufräumen** und fügen Sie den folgenden Code ein, um den Agent und den Thread zu löschen, wenn er nicht mehr benötigt wird.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. Überprüfen Sie den Code mithilfe der Kommentare, um zu verstehen, wie er:
    - Verbindet sich mit dem AI Foundry Projekt.
    - Lädt die Datendatei hoch und erstellt ein Code-Interpreter-Tool, das auf die Datei zugreifen kann.
    - Erstellt einen neuen Agent, der das Codedolmetschertool verwendet, und verfügt über explizite Anweisungen zum Analysieren der Daten und Erstellen von Diagrammen als .png Dateien.
    - Führt einen Thread mit einer Eingabeaufforderungsmeldung der benutzenden Person zusammen mit den zu analysierenden Daten aus.
    - Überprüft den Status der Ausführung, falls ein Fehler auftritt
    - Ruft die Nachrichten aus dem abgeschlossenen Thread ab und zeigt die letzte vom Agenten gesendete Nachricht an.
    - Zeigt die aufgezeichneten Unterhaltungen an.
    - Speichert jede generierte Datei.
    - Löscht den Agent und den Thread, wenn er nicht mehr benötigt wird.

1. Speichern Sie die Codedatei (*STRG+S*), wenn Sie fertig sind. Sie können auch den Code-Editor (*STRG+Q*) schließen. Möglicherweise möchten Sie ihn jedoch geöffnet lassen, falls Sie Änderungen an dem Code vornehmen müssen, den Sie hinzugefügt haben. Lassen Sie in beiden Fällen den Befehlszeilenbereich der Cloud Shell geöffnet.

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
    python agent.py
    ```
    
    Die Anwendung wird mit den Anmeldeinformationen für Ihre authentifizierte Azure-Sitzung ausgeführt, um eine Verbindung zu Ihrem Projekt herzustellen und den Agent zu erstellen und auszuführen.

1. Wenn Sie dazu aufgefordert werden, zeigen Sie die Daten an, die die Anwendung aus der Textdatei *data.txt* geladen hat. Geben Sie dann eine Eingabeaufforderung wie die folgende ein:

    ```
   What's the category with the highest cost?
    ```

    > **Tipp**: Wenn die App fehlschlägt, weil das Ratenlimit überschritten wird. Warten Sie einige Sekunden, und versuchen Sie es noch mal. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

1. Zeigen Sie die Antwort an. Geben Sie dann eine weitere Eingabeaufforderung ein, diesmal zur Anforderung eines Diagramms:

    ```
   Create a pie chart showing cost by category
    ```

    Der Agent sollte das Code-Interpreter-Tool bei Bedarf selektiv einsetzen, in diesem Fall zur Erstellung eines Diagramms auf der Grundlage Ihrer Anfrage.

1. Sie können die Unterhaltung fortsetzen, wenn Sie möchten. Der Thread ist *statusbehaftet*,er behält die aufgezeichneten Unterhaltungen bei, was bedeutet, dass der Agent den vollständigen Kontext für jede Antwort hat. Geben Sie `quit` ein, wenn Sie fertig sind.
1. Überprüfen Sie die Unterhaltungsnachrichten, die aus dem Thread abgerufen wurden, und die erzeugten Dateien.

1. Wenn die Anwendung beendet ist, verwenden Sie den Befehl **download** der Cloud Shell, um jede .png-Datei herunterzuladen, die im App-Ordner gespeichert wurde. Zum Beispiel:

    ```
   download ./<file_name>.png
    ```

    Der Downloadbefehl erstellt unten rechts im Browser einen Popuplink, den Sie auswählen können, um die Datei herunterzuladen und zu öffnen.

## Zusammenfassung

In dieser Übung haben Sie das Azure AI Agent-Dienst SDK verwendet, um eine Client-Anwendung zu erstellen, die einen KI-Agent verwendet. Der Agent verwendet das integrierte Code-Interpreter-Tool, um dynamischen Code auszuführen, der Bilder erstellt.

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
