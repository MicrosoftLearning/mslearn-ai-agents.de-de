---
lab:
  title: Verbinden von KI-Agents mit einem MCP-Remoteserver
  description: 'Hier erfahren Sie, wie Sie Model Context Protocol-Tools in KI-Agents integrieren.'
---

# Verbinden von KI-Agents mit Tools mithilfe des MCP (Model Context Protocol)

In dieser Übung erstellen Sie einen Agent, der eine Verbindung mit einem in der Cloud gehosteten MCP-Server herstellt. Der Agent verwendet eine KI-gestützte Suche, um Programmierenden zu helfen, in Echtzeit genaue Antworten aus der offiziellen Dokumentation von Microsoft zu finden. Dies ist hilfreich beim Erstellen von Assistenten, die Programmierende mit aktuellen Anleitungen zu Tools wie Azure, .NET und Microsoft 365 unterstützen. Der Agent verwendet das bereitgestellte Tool `microsoft_docs_search`, um die Dokumentation abzufragen und relevante Ergebnisse zurückzugeben.

> **Tipp**: Der in dieser Übung verwendete Code basiert auf dem MCP-Beispielsupportrepository für den Azure AI Agent Service. Weitere Informationen finden Sie in den [Azure OpenAI-Demos](https://github.com/retkowsky/Azure-OpenAI-demos/blob/main/Azure%20Agent%20Service/9%20Azure%20AI%20Agent%20service%20-%20MCP%20support.ipynb) oder unter [Herstellen einer Verbindung mit Model Context Protocol-Servern](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/model-context-protocol).

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
    - **Region**: *Wählen Sie einen beliebigen Standort aus, an dem KI Services unterstützt wird***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Klicken Sie auf **Erstellen**, und warten Sie, bis das Projekt erstellt wird.
1. Wenn Sie dazu aufgefordert werden, stellen Sie ein **gpt-4o**-Modell entweder mithilfe der Bereitstellungsoption *Globaler Standard* oder *Standard* bereit (je nach Kontingentverfügbarkeit).

    >**Hinweis:** Wenn ein Kontingent verfügbar ist, kann ein GPT-4o-Basismodell automatisch bereitgestellt werden, wenn Sie Ihren Agent und Ihr Projekt erstellen.

1. Wenn Ihr Projekt erstellt wird, wird der Agents-Playground geöffnet.

1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    ![Screenshot einer Projektübersichtsseite von Azure AI Foundry.](./Media/ai-foundry-project.png)

1. Kopieren Sie den Wert **Azure AI Foundry-Projekt-Endgerät** in einen Notizblock, da Sie ihn für die Verbindung mit Ihrem Projekt in einer Clientanwendung benötigen.

## Entwickeln eines Agents, der MCP-Funktionstools verwendet

Nachdem Sie Ihr Projekt in AI Foundry erstellt haben, entwickeln wir nun eine App, die einen KI-Agent in einen MCP-Server integriert.

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
   cd ai-agents/Labfiles/03c-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt --pre azure-ai-projects mcp
    ```

    >**Hinweis:** Sie können alle während der Installation der Bibliothek angezeigten Warn- oder Fehlermeldungen ignorieren.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert aus der Projektseite **Übersicht** im Azure AI Foundry-Portal), und stellen Sie sicher, dass die Variable MODEL_DEPLOYMENT_NAME auf den Namen Ihrer Modellbereitstellung festgelegt ist (dies sollte *gpt-4o* sein).

1. Nachdem Sie den Platzhalter ersetzt haben, speichern Sie Ihre Änderungen mit dem Befehl **STRG+S** und schließen Sie den Code-Editor mit dem Befehl **STRG+Q**, wobei die Cloud Shell-Befehlszeile geöffnet bleibt.

### Verbinden eines Azure KI-Agents mit einem MCP-Remoteserver

In dieser Aufgabe stellen Sie eine Verbindung mit einem MCP-Remoteserver her, bereiten den KI-Agent vor und führen einen Benutzerprompt aus.

1. Suchen Sie den Kommentar **Referenzen hinzufügen**, und fügen Sie den folgenden Code hinzu, um die Klassen zu importieren:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import McpTool
    ```

1. Suchen Sie den Kommentar **Mit dem Agents-Client verbinden**, und fügen Sie den folgenden Code hinzu, um mit den aktuellen Azure-Anmeldeinformationen eine Verbindung mit dem Azure KI-Projekt herzustellen.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. Fügen Sie unter dem Kommentar **MCP-Agent-Tool initialisieren** den folgenden Code hinzu:

    ```python
   # Initialize agent MCP tool
   mcp_tool = McpTool(
       server_label=mcp_server_label,
       server_url=mcp_server_url,
   )
    ```

    Dieser Code stellt eine Verbindung mit dem MCP-Remoteserver der Microsft Learn Dokumentation her. Dies ist ein in der Cloud gehosteter Dienst, mit dem Clients direkt auf vertrauenswürdige und aktuelle Informationen in der offiziellen Microsoft-Dokumentation zugreifen können.

1. Fügen Sie unter dem Kommentar **Neuen Agent mit den MCP-Tooldefinitionen erstellen** den folgenden Code hinzu:

    ```python
   # Create a new agent with the mcp tool definitions
   agent = agents_client.create_agent(
       model=model_deployment,
       name="my-mcp-agent",
       instructions="""
        You have access to an MCP server called `microsoft.docs.mcp` - this tool allows you to 
        search through Microsoft's latest official documentation. Use the available MCP tools 
        to answer questions and perform tasks.""",
       tools=mcp_tool.definitions,
   )
    ```

    In diesem Code geben Sie Anweisungen für den Agent und stellen ihm die MCO-Tooldefinitionen zur Verfügung.

1. Suchen Sie den Kommentar **Kommunikationsthread erstellen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Create thread for communication
   thread = agents_client.threads.create()
   print(f"Created thread, ID: {thread.id}")
    ```

1. Suchen Sie den Kommentar **Nachricht im Thread erstellen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Create a message on the thread
   message = agents_client.messages.create(
       thread_id=thread.id,
       role="user",
       content="Give me the Azure CLI commands to create an Azure Container App with a managed identity.",
   )
   print(f"Created message, ID: {message.id}")
    ```

1. Fügen Sie unter dem Kommentar **MCP-Toolheader aktualisieren** den folgenden Code hinzu:

    ```python
   # Update mcp tool headers
   mcp_tool.update_headers("SuperSecret", "123456")
    ```

1. Suchen Sie den Kommentar **Genehmigungsmodus festlegen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Set approval mode
   mcp_tool.set_approval_mode("never")
    ```

1. Suchen Sie den Kommentar **Agent-Ausführung im Thread mit MCP-Tools erstellen verarbeiten**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Create and process agent run in thread with MCP tools
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id, tool_resources=mcp_tool.resources)
   print(f"Created run, ID: {run.id}")
    ```
    
    Der KI-Agent ruft automatisch die verbundenen MCP-Tools auf, um die Promptanforderung zu verarbeiten. Zur Veranschaulichung dieses Prozesses gibt der Code unter dem Kommentar **Ausführungsschritte und Toolaufrufe anzeigen** alle über den MCP-Server aufgerufenen Tools aus.

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
   python client.py
    ```

    Etwa folgende Ausgabe sollte angezeigt werden:

    ```
    Created agent, ID: <<agent-id>>
    MCP Server: mslearn at https://learn.microsoft.com/api/mcp
    Created thread, ID: <<thread-id>>
    Created message, ID: <<message-id>>
    Created run, ID: <<run-id>>
    Run completed with status: RunStatus.COMPLETED
    Step <<step1-id>> status: completed

    Step <<step2-id>> status: completed
    MCP Tool calls:
        Tool Call ID: <<tool-call-id>>
        Type: mcp
        Type: microsoft_docs_search


    Conversation:
    --------------------------------------------------
    ASSISTANT: You can use Azure CLI to create an Azure Container App with a managed identity (either system-assigned or user-assigned). Below are the relevant commands and workflow:

    ---

    ### **1. Create a Resource Group**
    ```azurecli
    az group create --name myResourceGroup --location eastus
    ```

    {{continued...}}

    Mithilfe dieser Schritte können Sie eine Azure Container Apps-Instanz entweder mit systemseitig zugewiesenen oder benutzerseitig zugewiesenen verwalteten Identitäten bereitstellen, um eine nahtlose mit anderen Azure-Diensten durchzuführen.
    --------------------------------------------------
    BENUTZER: Nenne mir die Azure CLI Befehle zum Erstellen einer Azure Container Apps-Instanz mit einer verwalteten Identität.
    --------------------------------------------------
    Gelöschter Agent
    ```

    Notice that the agent was able to invoke the MCP tool `microsoft_docs_search` automatically to fulfill the request.

## Clean up

Now that you've finished the exercise, you should delete the cloud resources you've created to avoid unnecessary resource usage.

1. Open the [Azure portal](https://portal.azure.com) at `https://portal.azure.com` and view the contents of the resource group where you deployed the hub resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.
