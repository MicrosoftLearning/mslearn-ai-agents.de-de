# Verbinden von KI-Agents mit Tools mithilfe von Model Context Protocol (MCP)

In dieser Übung erstellen Sie einen Agent, der eine Verbindung mit einem MCP-Server herstellen und automatisch aufrufbare Funktionen entdecken kann.

Sie erstellen einen einfachen Agent zur Bestandsbewertung für einen Kosmetikhändler. Mithilfe des MCP-Servers kann der Agent Informationen zum Bestand abrufen und Vorschläge zur Nachbestellung oder zum Ausverkauf machen.

> **Tipp**: Der in dieser Übung verwendete Code basiert auf Azure AI Foundry und MCP-SDKs für Python. Sie können ähnliche Lösungen mithilfe der SDKs für Microsoft .NET entwickeln. Ausführliche Informationen finden Sie unter [Azure AI Foundry SDK-Clientbibliotheken](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) und [MCP C# SDK](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html).

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
   cd ai-agents/Labfiles/03d-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    Die bereitgestellten Dateien enthalten den Client- und Serveranwendungscode. Model Context Protocol bietet eine standardisierte Möglichkeit, KI-Modelle mit verschiedenen Datenquellen und Tools zu verbinden. Wir trennen `client.py` und `server.py`, um die Agent-Logik und Tooldefinitionen modular zu halten und die reale Architektur zu simulieren. 
    
    `server.py` definiert die Tools, die der Agent verwenden kann, und simuliert Back-End-Dienste oder Geschäftslogik. 
    `client.py` übernimmt die Einrichtung des KI-Agents, Benutzerprompts und das Aufrufen der Tools bei Bedarf.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**Hinweis:** Sie können alle während der Installation der Bibliothek angezeigten Warn- oder Fehlermeldungen ignorieren.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert aus der Projektseite **Übersicht** im Azure AI Foundry-Portal), und stellen Sie sicher, dass die Variable MODEL_DEPLOYMENT_NAME auf den Namen Ihrer Modellbereitstellung festgelegt ist (dies sollte *gpt-4o* sein).

1. Nachdem Sie den Platzhalter ersetzt haben, speichern Sie Ihre Änderungen mit dem Befehl **STRG+S** und schließen Sie den Code-Editor mit dem Befehl **STRG+Q**, wobei die Cloud Shell-Befehlszeile geöffnet bleibt.

### Implementieren eines MCP-Servers

Ein MCP-Server (Model Context Protocol) ist eine Komponente, die aufrufbare Tools hostet. Diese Tools sind Python-Funktionen, die für KI-Agents verfügbar gemacht werden können. Wenn Tools mit `@mcp.tool()`kommentiert werden, werden sie für den Client auffindbar, sodass ein KI-Agent sie während einer Unterhaltung oder Aufgabe dynamisch aufrufen kann. In dieser Aufgabe fügen Sie einige Tools hinzu, mit denen der Agent Inventurprüfungen durchführen kann.

1. Geben Sie den folgenden Befehl ein, um die Codedatei zu bearbeiten, die für Ihren Funktionscode bereitgestellt wurde:

    ```
   code server.py
    ```

    In dieser Codedatei definieren Sie die Tools, mit denen der Agent einen Back-End-Dienst für den Einzelhandel simulieren kann. Beachten Sie den Serversetupcode am Anfang der Datei. Es wird `FastMCP`verwendet, um schnell eine MCP-Serverinstanz namens „Bestand“ einzurichten. Dieser Server hostet die von Ihnen definierten Tools und macht sie für den Agent während des Labs zugänglich.

1. Suchen Sie den Kommentar **Ein Inventurprüfungstool hinzufügen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    Dieses Wörterbuch stellt eine Beispielinventur dar. Die `@mcp.tool()`-Anmerkung ermöglicht es dem LLM, Ihre Funktion zu entdecken. 

1. Suchen Sie den Kommentar **Ein Tool für wöchentlichen Verkaufs hinzufügen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. Speichern Sie die Datei (*CTRL+S*).

### Implementieren eines MCP-Clients

Ein MCP-Client ist die Komponente, die eine Verbindung mit dem MCP-Server herstellt, um Tools zu entdecken und aufzurufen. Sie können sich dies als Brücke zwischen dem Agent und den vom Server gehosteten Funktionen vorstellen, die eine dynamische Toolnutzung als Reaktion auf Benutzerprompts ermöglicht.

1. Geben Sie den folgenden Befehl ein, um mit der Bearbeitung des Clientcodes zu beginnen.

    ```
   code client.py
    ```

    > **Tipp**: Achten Sie beim Hinzufügen von Code zur Codedatei darauf, den richtigen Einzug beizubehalten.

1. Suchen Sie den Kommentar **Referenzen hinzufügen**, und fügen Sie den folgenden Code hinzu, um die Klassen zu importieren:

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. Suchen Sie den Kommentar **MCP-Server starten**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    In einem Standardproduktionssetup wird der Server getrennt vom Client ausgeführt. Für dieses Lab ist der Client jedoch dafür verantwortlich, den Server mithilfe des standardmäßigen Eingabe-/Ausgabe-Transports zu starten. Dadurch wird ein einfacher Kommunikationskanal zwischen den beiden Komponenten erstellt, und das Setup der lokalen Entwicklung wird vereinfacht.

1. Suchen Sie den Kommentar **Eine MCP-Clientsitzung erstellen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    Dadurch wird eine neue Clientsitzung mithilfe der Eingabe- und Ausgabestreams aus dem vorherigen Schritt erstellt. Durch das Aufrufen von `session.initialize` wird die Sitzung vorbereitet, um Tools zu entdecken und aufzurufen, die auf dem MCP-Server registriert sind.

1. Fügen Sie unter dem Kommentar **Verfügbare Tools auflisten** den folgenden Code hinzu, um zu überprüfen, ob der Client eine Verbindung mit dem Server hergestellt hat:

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    Ihre Clientsitzung kann nun mit Ihrem Azure KI-Agent verwendet werden.

### Verbinden der MCP-Tools mit Ihrem Agent

In dieser Aufgabe bereiten Sie den KI-Agent vor, nehmen Benutzerprompts an und rufen die Funktionstools auf.

1. Suchen Sie den Kommentar **Mit dem Agents-Client verbinden** und fügen Sie den folgenden Code hinzu, um mit den aktuellen Azure-Anmeldeinformationen eine Verbindung zum Azure KI-Projekt herzustellen.

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

1. Fügen Sie unter dem Kommentar **Auf dem Server verfügbare Tools auflisten** den folgenden Code hinzu:

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. Fügen Sie unter dem Kommentar **Eine Funktion für jedes Tool erstellen** den folgenden Code hinzu:

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    Dieser Code bindet die auf dem MCP-Server verfügbaren Tools dynamisch ein, sodass sie vom KI-Agent aufgerufen werden können. Jedes Tool wird in eine asynchrone Funktion umgewandelt und dann in ein `FunctionTool`-Paket aufgenommen, das der Agent verwenden kann.

1. Suchen Sie den Kommentar **Den Agent erstellen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. Suchen Sie den Kommentar **Automatischen Funktionsaufruf aktivieren**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. Fügen Sie unter dem Kommentar **Einen Thread für die Chatsitzung erstellen** den folgenden Code hinzu:

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. Suchen Sie den Kommentar **Den Prompt aufrufen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. Suchen Sie den Kommentar **Das entsprechende Funktionstool abrufen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    Dieser Code verwendet die Informationen aus dem Toolaufruf des Agent-Threads. Der Funktionsname und die Argumente werden abgerufen und zum Aufrufen der entsprechenden Funktion verwendet.

1. Fügen Sie unter dem Kommentar **Den Ausgabetext anfügen** den folgenden Code hinzu:

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. Fügen Sie unter dem Kommentar **Die Ausgabe des Toolaufrufs übermitteln** den folgenden Code hinzu:

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    Dieser Code signalisiert dem Agent-Thread, dass die erforderliche Aktion abgeschlossen ist, und aktualisiert die Toolaufrufausgaben.

1. Suchen Sie den Kommentar **Die Antwort anzeigen**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

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

1. Wenn Sie dazu aufgefordert werden, geben Sie eine Abfrage ein, wie etwa:

    ```
   What are the current inventory levels?
    ```

    > **Tipp**: Wenn die App fehlschlägt, weil das Ratenlimit überschritten wird. Warten Sie einige Sekunden, und versuchen Sie es noch mal. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

    Etwa folgende Ausgabe sollte angezeigt werden:

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

1. Sie können die Unterhaltung fortsetzen, wenn Sie möchten. Der Thread ist *statusbehaftet*,er behält die aufgezeichneten Unterhaltungen bei, was bedeutet, dass der Agent den vollständigen Kontext für jede Antwort hat. 

    Geben Sie beispielsweise folgende Prompts ein:

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    Geben Sie `quit` ein, wenn Sie fertig sind.

## Bereinigen

Nachdem Sie die Übung beendet haben, sollten Sie die von Ihnen erstellten Cloud-Ressourcen löschen, um eine unnötige Ressourcennutzung zu vermeiden.

1. Öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Hub-Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
