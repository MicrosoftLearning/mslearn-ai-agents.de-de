---
lab:
  title: Entwickeln einer Multi-Agent-Lösung
  description: 'Erfahren Sie, wie Sie mehrere Agents für die Zusammenarbeit mithilfe des semantischen Kernel-SDKs konfigurieren.'
---

# Entwickeln einer Multi-Agent-Lösung

In dieser Übung erstellen Sie ein Projekt, das zwei KI-Agents mithilfe des Semantic Kernel SDK orchestriert. Ein *Incident Manager*-Agent analysiert die Dienstprotokolldateien auf Probleme. Wenn ein Problem gefunden wird, empfiehlt der Incident Manager eine Lösungsmaßnahme, und ein *DevOps Assistant*-Agent nimmt die Empfehlung entgegen, ruft die Korrekturfunktion auf und führt die Lösung durch. Der Incident Manager Agent überprüft dann die aktualisierten Protokolle, um sicherzustellen, dass die Lösung erfolgreich war.

Für diese Übung werden vier Stichproben-Protokolldateien bereitgestellt. Der Agent-Code des DevOps Assistant aktualisiert die Beispielprotokolldateien nur mit einigen Beispielprotokollmeldungen.

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

Jetzt sind Sie bereit, ein generatives KI-Sprachmodell zur Unterstützung Ihrer Agents einzusetzen.

1. Wählen Sie im linken Fensterbereich für Ihr Projekt im Abschnitt **Meine Assets** die Seite **Modelle + Endpunkte**.
1. Wählen Sie auf der Seite **Modelle + Endpunkte** auf der Registerkarte **Modellbereitstellungen** im Menü **+ Modell bereitstellen** die Option **Basismodell bereitstellen**.
1. Suchen Sie das Modell **gpt-4o** in der Liste, wählen Sie es aus und bestätigen Sie es.
1. Stellen Sie das Modell mit den folgenden Einstellungen bereit, indem Sie **Anpassen** in den Bereitstellungsdetails wählen:
    - **Bereitstellungsname:***Ein eindeutiger Name für die Modellimplementierung*
    - **Bereitstellungstyp**: Globaler Standard
    - **Automatische Versionsaktualisierung**: Aktiviert
    - **Modellversion**: *Wählen Sie die neueste verfügbare Version aus.*
    - **Verbundene AI-Ressource**: *Wählen Sie Ihre Azure OpenAI-Ressourcenverbindung*
    - **Tokens pro Minute Ratenlimit (Tausende)**: 60K *(oder das in Ihrem Abonnement verfügbare Maximum, wenn weniger als 60K)*
    - **Inhaltsfilter**: StandardV2 

    > **Hinweis:** Durch das Verringern des TPM wird die Überlastung des Kontingents vermieden, das in dem von Ihnen verwendeten Abonnement verfügbar ist. 60.000 TPM sollten für die in dieser Übung verwendeten Daten ausreichend sein. Wenn Ihr verfügbares Kontingent darunter liegt, können Sie die Übung zwar abschließen, müssen aber möglicherweise warten und die Prompts erneut senden, wenn das Kontingent überschritten wird.

1. Warten Sie, bis die Bereitstellung abgeschlossen ist.

## Erstellen einer KI-Agent-Client-App

Jetzt können Sie eine Client-App erstellen, die einen Agent sowie eine benutzerdefinierte Funktion definiert. Einige Code werden für Sie in einem Github Repository bereitgestellt.

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
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    Die bereitgestellten Dateien enthalten Anwendungscode und eine Datei für Einstellungen der Konfiguration.

### Konfigurieren der Anwendungseinstellungen

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **Hinweis**: Die Installation von *semantic-kernel[azure]* installiert automatisch eine zum semantischen Kernel kompatible Version von *azure-ai-projects*.

1. Geben Sie den folgenden Befehl ein, um die mitgelieferte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_connection_string** mit der Verbindungszeichenfolge für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal) und den Platzhalter **your_model_deployment** mit dem Namen, den Sie Ihrer gpt-4o-Modell-Bereitstellung zugewiesen haben.

1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Erstellen von KI-Agents

Jetzt können Sie die Agents für Ihre Multi-Agent-Lösung erstellen! Legen wir los.

1. Geben Sie den folgenden Befehl ein, um die **agent_chat.py** Datei zu bearbeiten:

    ```
   code agent_chat.py
    ```

1. Überprüfen Sie den Code in der Datei und stellen Sie fest, dass er Folgendes enthält:
    - Konstanten, die die Namen und Anweisungen für Ihre beiden Agents festlegen.
    - Eine **Haupt-** Funktion, in der der größte Teil des Codes zur Implementierung Ihrer Multi-Agent-Lösung eingefügt wird.
    - Eine **SelectionStrategy** Klasse, die Sie verwenden werden, um die Logik zu implementieren, die erforderlich ist, um zu bestimmen, welcher Agent für jede Runde im Gespräch ausgewählt werden soll.
    - Eine **ApprovalTerminationStrategy** Klasse, die Sie verwenden werden, um die Logik zu implementieren, die benötigt wird, um zu bestimmen, wann das Gespräch beendet werden soll.
    - Eine **DevopsPlugin**-Klasse, die Funktionen zur Durchführung von Devops-Operationen enthält.
    - Eine **LogFilePlugin**-Klasse, die Funktionen zum Lesen und Schreiben von Protokolldateien enthält.

    Zunächst erstellen Sie den Agenten *Incident Manager*, der die Service-Protokolldateien analysiert, potenzielle Probleme identifiziert und bei Bedarf Lösungsmaßnahmen empfiehlt oder Probleme eskaliert.

1. Beachten Sie die Zeichenfolge **INCIDENT_MANAGER_INSTRUCTIONS**. Dies sind die Anweisungen für Ihren Agent.

1. Suchen Sie in der Funktion **main** den Kommentar **Erstellen Sie den Incident Manager Agent auf dem Azure AI Agent-Dienst**, und fügen Sie den folgenden Code hinzu, um einen Azure KI Agent zu erstellen.

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    Dieser Code erstellt die Agentdefinition auf Ihrem Azure AI Project-Client.

1. Suchen Sie den Kommentar **Erstellen eines semantischen Kernel-Agents für den Azure AI Incident Manager-Agent**, und fügen Sie den folgenden Code hinzu, um einen Semantic Kernel agent basierend auf der Azure AI Agent-Definition zu erstellen.

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    Dieser Code erstellt den semantischen Kernel-Agent mit Zugriff auf logFilePlugin****. Mit diesem Plug-In kann der Agent den Inhalt der Protokolldatei lesen.

    Erstellen wir nun den zweiten Agent, der auf Probleme reagiert und DevOps-Operationen zu deren Behebung durchführt.

1. Achten Sie am Anfang der Codedatei auf die Zeichenfolge **DEVOPS_ASSISTANT_INSTRUCTIONS**. Dies sind die Anweisungen, die Sie dem neuen DevOps-Assistenten geben werden.

1. Suchen Sie den Kommentar **Create the devops agent on the Azure AI agent service** und fügen Sie den folgenden Code hinzu, um eine Azure AI Agent-Definition zu erstellen:
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. Suchen Sie den Kommentar **Erstellen eines semantischen Kernel-Agents für den Azure-AI-Agent von devops**, und fügen Sie den folgenden Code hinzu, um einen Semantic Kernel agent basierend auf der Azure AI Agent-Definition zu erstellen.
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    Mit **DevopsPlugin** kann der Agent Devops-Aufgaben simulieren, z. B. den Dienst neu starten oder eine Transaktion zurücksetzen.

### Definieren von Gruppenchatstrategien

Nun müssen Sie die Logik bereitstellen, mit der bestimmt wird, welcher Agent für die nächste Gesprächsrunde ausgewählt werden soll und wann das Gespräch beendet werden soll.

Beginnen wir mit der **SelectionStrategy**, die angibt, welcher Agent den nächsten Schritt ausführen soll.

1. In der Klasse **SelectionStrategy** (unterhalb der Funktion **main**) finden Sie den Kommentar **Wählen Sie den nächsten Agent aus, der im Chat an der Reihe sein soll**, und fügen Sie den folgenden Code hinzu, um eine Auswahlfunktion zu definieren:

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    Dieser Code wird bei jedem Zug ausgeführt, um zu bestimmen, welcher Agent antworten soll, und überprüft den Chatverlauf, um zu sehen, wer zuletzt geantwortet hat.

    Nun wollen wir die Klasse **ApprovalTerminationStrategy** implementieren, um zu signalisieren, wann das Ziel erreicht ist und das Gespräch beendet werden kann.

1. Suchen Sie in der Klasse **ApprovalTerminationStrategy** den Kommentar **Beenden Sie den Chat, wenn der Agent angezeigt hat, dass keine Aktion erforderlich ist**, und fügen Sie den folgenden Code hinzu, um die Beendigungsfunktion zu definieren:

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    Der Kernel ruft diese Funktion nach der Antwort des Agents auf, um festzustellen, ob die Abschlusskriterien erfüllt sind. In diesem Fall wird das Ziel erreicht, wenn der Vorfallmanager mit „Keine Aktion erforderlich“ reagiert. Dieser Ausdruck wird in den Anweisungen des Incident Managers definiert.

### Implementieren des Gruppenchats

Nun, da Sie zwei Agenten haben und Strategien, die ihnen helfen, sich abzuwechseln und einen Chat zu beenden, können Sie den Gruppenchat implementieren.

1. Gehen Sie zurück in die Hauptfunktion, suchen Sie den Kommentar **Hinzufügen der Agents zu einem Gruppenchat mit einer benutzerdefinierten Beendigungs- und Auswahlstrategie**, und fügen Sie den folgenden Code hinzu, um den Gruppenchat zu erstellen:

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    In diesem Code erstellen Sie ein Agent-Gruppen-Chat-Objekt mit dem Incident Manager und den Devops-Agents. Außerdem definieren Sie die Beendigungs- und Auswahlstrategien für den Chat. Beachten Sie, dass die **ApprovalTerminationStrategy** nur an den Incident Manager Agent und nicht an den Devops Agent gebunden ist. Damit ist der Incident Manager Agent dafür verantwortlich, das Ende des Chats zu signalisieren. Die **SelectionStrategy** beinhaltet alle Agenten, die im Chat an der Reihe sein sollen.

    Beachten Sie, dass das Flag zum automatischen Zurücksetzen den Chat beim Beenden automatisch löscht. Auf diese Weise kann der Agent die Dateien weiterhin analysieren, ohne dass das Chatverlaufsobjekt zu viele unnötige Token verwendet. 

1. Suchen Sie den Kommentar **Anfügen der aktuellen Protokolldatei an den Chat**, und fügen Sie den folgenden Code hinzu, um dem Chat den zuletzt gelesenen Protokolldateitext hinzuzufügen:

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. Suchen Sie den Kommentar **Aufruf einer Antwort von den Agents**, und fügen Sie den folgenden Code hinzu, um den Gruppenchat aufzurufen:

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    Dies ist der Code, der den Chat auslöst. Da der Text der Protokolldatei als Nachricht hinzugefügt wurde, bestimmt die Auswahlstrategie, welcher Agent sie lesen und beantworten soll, und dann wird die Konversation zwischen den Agents fortgesetzt, bis die Bedingungen der Beendigungsstrategie erfüllt sind oder die maximale Anzahl von Iterationen erreicht ist.

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
   python agent_chat.py
    ```

    Etwa folgende Ausgabe sollte angezeigt werden:

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **Hinweis**: Die App enthält einen Code, der zwischen der Verarbeitung jeder Protokolldatei wartet, um das Risiko einer Überschreitung des TPM-Ratenlimits zu verringern, sowie eine Ausnahmebehandlung für den Fall, dass dies dennoch geschieht. Wenn in Ihrem Abonnement nicht genügend Kontingent verfügbar ist, kann das Modell möglicherweise nicht reagieren.

1. Überprüfen Sie, ob die Protokolldateien im Ordner **logs** mit den Auflösungsmeldungen von DevopsAssistant aktualisiert werden.

    An log1.log sollten zum Beispiel die folgenden Protokollmeldungen angehängt werden:

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## Zusammenfassung

In dieser Übung haben Sie den Azure AI Agent-Dienst und das semantische Kernel SDK verwendet, um AI Incident- und Devops-Agents zu erstellen, die automatisch Probleme erkennen und Lösungen anwenden können. Gut gemacht!

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.

1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.

1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.