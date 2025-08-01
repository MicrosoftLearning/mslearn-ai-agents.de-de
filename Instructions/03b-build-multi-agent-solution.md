---
lab:
  title: Entwickeln einer Multi-Agent-Lösung mit Azure AI Foundry
  description: 'Erfahren Sie, wie Sie mehrere Agents für die Zusammenarbeit mit Azure KI Foundry Agent Service konfigurieren.'
---

# Entwickeln einer Multi-Agent-Lösung

In dieser Übung erstellen Sie ein Projekt, das mehrere KI-Agents mithilfe des Azure AI Foundry Agent Service koordiniert. Sie entwerfen eine KI-Lösung, die bei der Ticket-Einstufung hilft. Die verbundenen Agents bewerten die Priorität des Tickets, schlagen eine Teamzuweisung vor und bestimmen den Aufwand, der zum Abschließen des Tickets erforderlich ist. Legen wir los.

> **Tipp**: Der in dieser Übung verwendete Code basiert auf dem Azure AI Foundry SDK für Python. Sie können ähnliche Lösungen mithilfe der SDKs für Microsoft .NET, JavaScript und Java entwickeln. Ausführliche Informationen finden Sie unter [Azure AI Foundry SDK-Clientbibliotheken](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview).

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

    > **Hinweis**: Die Standard-TPM-Einstellung für dieses Modell ist möglicherweise für diese Übung zu niedrig. Ein niedrigerer TPM-Wert hilft dabei, eine Übernutzung des in Ihrem Abonnement verfügbaren Kontingents zu vermeiden. 

1. Wählen Sie im Navigationsbereich links **Modelle und Endgeräte** und wählen Sie Ihre **gpt-4o**-Bereitstellung aus.

1. Wählen Sie **Bearbeiten**, und erhöhen Sie die **Token pro Minute-Begrenzung**.

   > **HINWEIS**: 40.000 TPM sollten für die in dieser Übung verwendeten Daten ausreichend sein. Wenn Ihr verfügbares Kontingent darunter liegt, können Sie die Übung zwar abschließen, müssen aber möglicherweise warten und die Prompts erneut senden, wenn das Kontingent überschritten wird.

1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    ![Screenshot einer Projektübersichtsseite von Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Kopieren Sie den Wert **Azure AI Foundry-Projekt-Endgerät** in einen Notizblock, da Sie ihn für die Verbindung mit Ihrem Projekt in einer Clientanwendung benötigen.

## Erstellen einer KI-Agent-Client-App

Jetzt können Sie eine Client-App erstellen, die die Agents und Anweisungen definiert. Einige Code werden für Sie in einem Github Repository bereitgestellt.

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

    > **Tipp**: Wenn Sie Befehle in die Cloud-Shell eingeben, kann die Ausgabe einen großen Teil des Bildschirmpuffers in Anspruch nehmen und der Cursor in der aktuellen Zeile kann verdeckt werden. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

1. Wenn das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Arbeitsverzeichnis in den Ordner, der die Codedateien enthält, zu ändern und alle aufzulisten.

    ```
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    Die bereitgestellten Dateien enthalten Anwendungscode und eine Datei für Einstellungen der Konfiguration.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. Geben Sie den folgenden Befehl ein, um die mitgelieferte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Code-Datei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrer gpt-4o-Modellbereitstellung zugewiesen haben.

1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Erstellen von KI-Agents

Jetzt können Sie die Agents für Ihre Multi-Agent-Lösung erstellen! Legen wir los.

1. Geben Sie den folgenden Befehl ein, um die Datei **agent_triage.py** zu bearbeiten:

    ```
   code agent_triage.py
    ```

1. Überprüfen Sie den Code in der Datei und beachten Sie, dass er Zeichenfolgen für jeden Agentnamen und Anweisungen enthält.

1. Suchen Sie den Kommentar **Referenzen hinzufügen** und fügen Sie den folgenden Code hinzu, um die benötigten Klassen zu importieren:

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
    from azure.identity import DefaultAzureCredential
    ```

1. Geben Sie unter dem Kommentar **Anweisungen für den primären Agent** den folgenden Code ein:

    ```python
    # Instructions for the primary agent
    triage_agent_instructions = """
    Triage the given ticket. Use the connected tools to determine the ticket's priority, 
    which team it should be assigned to, and how much effort it may take.
    """
    ```

1. Suchen Sie den Kommentar **Prioritäts-Agent für den Azure KI-Agent-Dienst erstellen**, und fügen Sie den folgenden Code hinzu, um einen Azure KI-Agent zu erstellen.

    ```python
    # Create the priority agent on the Azure AI agent service
    priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
    )
    ```

    Dieser Code erstellt die Agentdefinition auf Ihrem Azure AI Agents-Client.

1. Suchen Sie den Kommentar **Verbundenes Agent-Tool für den Prioritäts-Agent erstellen**, und fügen Sie den folgenden Code hinzu:

    ```python
    # Create a connected agent tool for the priority agent
    priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
    )
    ```

    Erstellen Sie nun die anderen Einstufungs-Agents.

1. Fügen Sie unter dem Kommentar **Team-Agent und verbundenes Tool erstellen** den folgenden Code hinzu:
    
    ```python
    # Create the team agent and connected tool
    team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
    )
    team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
    )
    ```

1. Fügen Sie unter dem Kommentar **Aufwands-Agent und verbundenes Tool erstellen** den folgenden Code hinzu:
    
    ```python
    # Create the effort agent and connected tool
    effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
    )
    effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
    )
    ```


1. Fügen Sie unter dem Kommentar **Create a main agent with the Connected Agent tools** den folgenden Code hinzu:
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="triage-agent",
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
    )
    ```

1. Suchen Sie den Kommentar **Create thread for the chat session** und fügen Sie den folgenden Code hinzu:
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. Fügen Sie unter dem Kommentar **Ticketprompt erstellen** den folgenden Code hinzu:
    
    ```python
    # Create the ticket prompt
    prompt = "Users can't reset their password from the mobile app."

    ```

1. Fügen Sie unter dem Kommentar **Send a prompt to the agent** den folgenden Code hinzu:
    
    ```python
    # Send a prompt to the agent
    message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
    )
    ```

1. Fügen Sie unter dem Kommentar **Create and process Agent run in thread with tools** den folgenden Code hinzu:
    
    ```python
    # Create and process Agent run in thread with tools
    print("Processing agent thread. Please wait.")
    run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    ```


1. Verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen in der Codedatei zu speichern. Sie können ihn geöffnet lassen (für den Fall, dass Sie den Code bearbeiten müssen, um Fehler zu beheben) oder den Befehl **STRG+Q** verwenden, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Bei Azure anmelden und die App ausführen

Jetzt können Sie Ihren Code ausführen und Ihre KI-Agents bei der Zusammenarbeit beobachten.

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
   az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry Hub enthält, wenn Sie dazu aufgefordert werden.

1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
   python agent_triage.py
    ```

    Etwa folgende Ausgabe sollte angezeigt werden:

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    Sie können versuchen, den Prompt mit einem anderen Ticketszenario zu ändern, um zu beobachten, wie die Agents zusammenarbeiten. Beispiel: „Untersuche gelegentliche 502-Fehler vom Suchendpunkt.“

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.

1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.

1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
