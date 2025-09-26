---
lab:
  title: Entwickeln einer Multi-Agent-Lösung mit Semantischem Kernel
  description: 'Erfahren Sie, wie Sie mehrere Agents für die Zusammenarbeit mithilfe des semantischen Kernel-SDKs konfigurieren.'
---

# Entwickeln einer Multi-Agent-Lösung

In dieser Übung üben Sie die Verwendung des Musters für sequenzielle Orchestrierungen im Semantic Kernel SDK. Sie erstellen eine einfache Pipeline mit drei Agents, die zusammenarbeiten, um Kundenfeedback zu verarbeiten und die nächsten Schritte vorzuschlagen. Sie erstellen die folgenden Agents:

- Der Zusammenfassungs-Agent komprimiert unformatiertes Feedback in einen kurzen, neutralen Satz.
- Der Klassifizierer-Agent kategorisiert das Feedback als „Positiv“, „Negativ“ oder „Featureanforderung“.
- Schließlich empfiehlt der Agent für empfohlene Aktionen einen geeigneten Nachverfolgungsschritt.

Sie erfahren, wie Sie das Semantic Kernel SDK verwenden, um ein Problem aufzuschlüsseln, es an die richtigen Agents zu leiten und umsetzbare Ergebnisse zu erzielen. Legen wir los.

> **Tipp**: Der in dieser Übung verwendete Code basiert auf dem Semantic Kernel SDK für Python. Sie können ähnliche Lösungen mithilfe der SDKs für Microsoft .NET und Java entwickeln. Ausführliche Informationen finden Sie unter [Unterstützte Sprachen für semantische Kernel](https://learn.microsoft.com/semantic-kernel/get-started/supported-languages).

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
    - **Region**: Wählen Sie die **empfohlene AI Foundry-Instanz aus***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Wählen Sie **Erstellen** und warten Sie, bis Ihr Projekt einschließlich der von Ihnen ausgewählten GPT-4-Modellbereitstellung erstellt wurde.

1. Wenn Ihr Projekt erstellt wird, wird der Chat-Playground automatisch geöffnet.

1. Wählen Sie im Navigationsbereich links **Modelle und Endgeräte** und wählen Sie Ihre **gpt-4o**-Bereitstellung aus.

1. Notieren Sie sich im Bereich **Setup** den Namen Ihrer Modellbereitstellung, der **gpt-4o** lauten sollte. Sie können dies überprüfen, indem Sie die Bereitstellung auf der Seite **Modelle und Endpunkte** anzeigen (öffnen Sie dazu einfach diese Seite im Navigationsbereich auf der linken Seite).
1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht**, um die Hauptseite Ihres Projekts anzuzeigen, die wie folgt aussieht:

    ![Screenshot eines Azure KI-Projekts im Azure AI Foundry-Portal.](./Media/ai-foundry-project.png)

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
   pip install python-dotenv azure-identity semantic-kernel --upgrade
    ```

    > **Hinweis:** Durch die Installation von *semantic-kernel* wird automatisch eine mit dem semantischen Kernel kompatible Version von *azure-ai-projects* installiert.

1. Geben Sie den folgenden Befehl ein, um die mitgelieferte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_openai_endpoint** durch den Azure OpenAI-Endpunkt für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal unter **Azure OpenAI**). Ersetzen Sie **your_openai_api_key** durch den API-Schlüssel für Ihr Projekt, und stellen Sie sicher, dass die Variable MODEL_DEPLOYMENT_NAME auf ihren Modellbereitstellungsnamen festgelegt ist (dieser sollte *gpt-4o* lauten).

1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie den Befehl **STRG+S**, um Ihre Änderungen zu speichern, und verwenden Sie dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

### Erstellen von KI-Agents

Jetzt können Sie die Agents für Ihre Multi-Agent-Lösung erstellen! Legen wir los.

1. Geben Sie den folgenden Befehl ein, um die Datei **agents.py** zu bearbeiten:

    ```
   code agents.py
    ```

1. Fügen Sie oben in der Datei unter dem Kommentar **Verweise hinzufügen** den folgenden Code hinzu, um auf die Namespaces in den Bibliotheken zu verweisen, die Sie zum Implementieren Ihres Agents benötigen:

    ```python
   # Add references
   import asyncio
   from semantic_kernel.agents import Agent, ChatCompletionAgent, SequentialOrchestration
   from semantic_kernel.agents.runtime import InProcessRuntime
   from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
   from semantic_kernel.contents import ChatMessageContent
    ```


1. Fügen Sie in der Funktion **get_agents** den folgenden Code unter dem Kommentar **Zusammenfassungs-Agent erstellen** hinzu:

    ```python
   # Create a summarizer agent
   summarizer_agent = ChatCompletionAgent(
       name="SummarizerAgent",
       instructions="""
       Summarize the customer's feedback in one short sentence. Keep it neutral and concise.
       Example output:
       App crashes during photo upload.
       User praises dark mode feature.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Fügen Sie unter dem Kommentar **Create a classifier agent** den folgenden Code hinzu:

    ```python
   # Create a classifier agent
   classifier_agent = ChatCompletionAgent(
       name="ClassifierAgent",
       instructions="""
       Classify the feedback as one of the following: Positive, Negative, or Feature request.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Fügen Sie unter dem Kommentar **Create a recommended action agent** den folgenden Code hinzu:

    ```python
   # Create a recommended action agent
   action_agent = ChatCompletionAgent(
       name="ActionAgent",
       instructions="""
       Based on the summary and classification, suggest the next action in one short sentence.
       Example output:
       Escalate as a high-priority bug for the mobile team.
       Log as positive feedback to share with design and marketing.
       Log as enhancement request for product backlog.
       """,
       service=AzureChatCompletion(),
   )
    ```

1. Fügen Sie unter dem Kommentar **Return a list of agents** den folgenden Code hinzu:

    ```python
   # Return a list of agents
   return [summarizer_agent, classifier_agent, action_agent]
    ```

    Die Reihenfolge der Agents in dieser Liste ist die Reihenfolge, in der sie während der Orchestrierung ausgewählt werden.

## Erstellen einer sequenziellen Orchestrierung

1. Suchen Sie in der **main-Funktion** den Kommentar **Initialize the input task**, und fügen Sie den folgenden Code hinzu:
    
    ```python
   # Initialize the input task
   task="""
   I tried updating my profile picture several times today, but the app kept freezing halfway through the process. 
   I had to restart it three times, and in the end, the picture still wouldn't upload. 
   It's really frustrating and makes the app feel unreliable.
   """
    ```

1. Fügen Sie unter dem Kommentar **Create a sequential orchestration** den folgenden Code hinzu, um eine sequenzielle Orchestrierung mit einem Antwortrückruf zu definieren:

    ```python
   # Create a sequential orchestration
   sequential_orchestration = SequentialOrchestration(
       members=get_agents(),
       agent_response_callback=agent_response_callback,
   )
    ```

    Mit `agent_response_callback` können Sie die Antwort der einzelnen Agents während der Orchestrierung anzeigen.

1. Fügen Sie unter dem Kommentar **Create a runtime and start it** den folgenden Code hinzu:

    ```python
   # Create a runtime and start it
   runtime = InProcessRuntime()
   runtime.start()
    ```

1. Fügen Sie unter dem Kommentar **Invoke the orchestration with a task and the runtime** den folgenden Code hinzu:

    ```python
   # Invoke the orchestration with a task and the runtime
   orchestration_result = await sequential_orchestration.invoke(
       task=task,
       runtime=runtime,
   )
    ```

1. Fügen Sie unter dem Kommentar **Wait for the results** den folgenden Code hinzu:

    ```python
   # Wait for the results
   value = await orchestration_result.get(timeout=20)
   print(f"\n****** Task Input ******{task}")
   print(f"***** Final Result *****\n{value}")
    ```

    In diesem Code rufen Sie das Ergebnis der Orchestrierung ab und zeigen es an. Wenn die Orchestrierung nicht innerhalb des angegebenen Timeouts abgeschlossen ist, wird eine Timeoutausnahme ausgelöst.

1. Suchen Sie den Kommentar **Stop the runtime when idle**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Stop the runtime when idle
   await runtime.stop_when_idle()
    ```

    Beenden Sie nach Abschluss der Verarbeitung die Runtime, um Ressourcen zu bereinigen.

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
   python agents.py
    ```

    Etwa folgende Ausgabe sollte angezeigt werden:

    ```output
    # SummarizerAgent
    App freezes during profile picture upload, preventing completion.
    # ClassifierAgent
    Negative
    # ActionAgent
    Escalate as a high-priority bug for the development team.

    ****** Task Input ******
    I tried updating my profile picture several times today, but the app kept freezing halfway through the process.
    I had to restart it three times, and in the end, the picture still wouldn't upload.
    It's really frustrating and makes the app feel unreliable.

    ***** Final Result *****
    Escalate as a high-priority bug for the development team.
    ```

1. Optional können Sie versuchen, den Code mithilfe verschiedener Aufgabeneingaben auszuführen, z. B.:

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## Zusammenfassung

In dieser Übung haben Sie die sequenzielle Orchestrierung mit dem Semantic Kernel SDK geübt und mehrere Agents in einem einzelnen optimierten Workflow kombiniert. Gut gemacht!

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.

1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.

1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
