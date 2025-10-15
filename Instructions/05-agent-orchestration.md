---
lab:
  title: Entwickeln einer Multi-Agent-Lösung mit Microsoft Agent Framework
  description: 'Erfahren Sie, wie Sie mehrere Agents für die Zusammenarbeit mit dem Microsoft Agent Framework SDK konfigurieren'
---

# Entwickeln einer Multi-Agent-Lösung

In dieser Übung üben Sie die Verwendung des sequenziellen Orchestrierungsmusters im Microsoft Agent Framework SDK. Sie erstellen eine einfache Pipeline mit drei Agents, die zusammenarbeiten, um Kundenfeedback zu verarbeiten und die nächsten Schritte vorzuschlagen. Sie erstellen die folgenden Agents:

- Der Zusammenfassungs-Agent komprimiert unformatiertes Feedback in einen kurzen, neutralen Satz.
- Der Klassifizierer-Agent kategorisiert das Feedback als „Positiv“, „Negativ“ oder „Featureanforderung“.
- Schließlich empfiehlt der Agent für empfohlene Aktionen einen geeigneten Nachverfolgungsschritt.

Sie erfahren, wie Sie das Microsoft Agent Framework SDK verwenden, um ein Problem aufzuschlüsseln, es durch die richtigen Agents zu leiten und umsetzbare Ergebnisse zu erzielen. Legen wir los.

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
   pip install azure-identity agent-framework
    ```

1. Geben Sie den folgenden Befehl ein, um die mitgelieferte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_openai_endpoint** durch den Endpunkt für Ihr Projekt (kopiert aus der Seite **Übersicht** des Projekts im Azure AI Foundry-Portal). Ersetzen Sie den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrer gpt-4o-Modellimplementierung zugewiesen haben.

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
   from typing import cast
   from agent_framework import ChatMessage, Role, SequentialBuilder, WorkflowOutputEvent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity import AzureCliCredential
    ```

1. Nehmen Sie sich in der **main**-Funktion einen Moment Zeit, um die Agentenanweisungen zu überprüfen. Diese Anweisungen definieren das Verhalten jedes Agents in der Orchestrierung.

1. Fügen Sie den folgenden Code unter dem Kommentar **Create the chat client** hinzu:

    ```python
   # Create the chat client
   credential = AzureCliCredential()
   async with (
       AzureAIAgentClient(async_credential=credential) as chat_client,
   ):
    ```

    Beachten Sie, dass das **AzureCliCredential**-Objekt Ihrem Code die Authentifizierung bei Ihrem Azure-Konto ermöglicht. Das **AzureAIAgentClient**-Objekt enthält automatisch die Azure AI Foundry-Projekteinstellungen aus der ENV-Konfiguration.

1. Fügen Sie den folgenden Code unter dem Kommentar **Create agents** hinzu:

    (Achten Sie darauf, die Ebene des Einzugs beizubehalten)

    ```python
   # Create agents
   summarizer = chat_client.create_agent(
       instructions=summarizer_instructions,
       name="summarizer",
   )

   classifier = chat_client.create_agent(
       instructions=classifier_instructions,
       name="classifier",
   )

   action = chat_client.create_agent(
       instructions=action_instructions,
       name="action",
   )
    ```

## Erstellen einer sequenziellen Orchestrierung

1. Suchen Sie in der **main**-Funktion den Kommentar **Initialize the current feedback** und fügen Sie den folgenden Code hinzu:
    
    (Achten Sie darauf, die Ebene des Einzugs beizubehalten)

    ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
    ```

1. Fügen Sie unter dem Kommentar **Build a sequential orchestration** den folgenden Code hinzu, um eine sequenzielle Orchestrierung mit den von Ihnen definierten Agents zu definieren:

    ```python
   # Build sequential orchestration
   workflow = SequentialBuilder().participants([summarizer, classifier, action]).build()
    ```

    Die Agenten verarbeiten das Feedback in der Reihenfolge, in der sie der Orchestrierung hinzugefügt werden.

1. Fügen Sie den folgenden Code unter dem Kommentar **Run and collect outputs** hinzu:

    ```python
   # Run and collect outputs
   outputs: list[list[ChatMessage]] = []
   async for event in workflow.run_stream(f"Customer feedback: {feedback}"):
       if isinstance(event, WorkflowOutputEvent):
           outputs.append(cast(list[ChatMessage], event.data))
    ```

    Dieser Code führt die Orchestrierung aus und sammelt die Ausgabe von jedem der teilnehmenden Agents.

1. Fügen Sie den folgenden Code unter dem Kommentar **Display outputs** hinzu:

    ```python
   # Display outputs
   if outputs:
       for i, msg in enumerate(outputs[-1], start=1):
           name = msg.author_name or ("assistant" if msg.role == Role.ASSISTANT else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
    ```

    Dieser Code formatiert und zeigt die Nachrichten aus dem Workflow an, die Sie aus der Orchestrierung gesammelt haben.

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
    ------------------------------------------------------------
    01 [user]
    Customer feedback:
        I use the dashboard every day to monitor metrics, and it works well overall.
        But when I'm working late at night, the bright screen is really harsh on my eyes.
        If you added a dark mode option, it would make the experience much more comfortable.

    ------------------------------------------------------------
    02 [summarizer]
    User requests a dark mode for better nighttime usability.
    ------------------------------------------------------------
    03 [classifier]
    Feature request
    ------------------------------------------------------------
    04 [action]
    Log as enhancement request for product backlog.
    ```

1. Optional können Sie versuchen, den Code mithilfe verschiedener Feedbackeingaben auszuführen, z. B.:

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## Zusammenfassung

In dieser Übung haben Sie die sequenzielle Orchestrierung mit dem Microsoft Agent Framework SDK geübt und mehrere Agents in einem einzigen optimierten Workflow kombiniert. Gut gemacht!

## Bereinigen

Wenn Sie die Erkundung des Azure AI Agent-Dienstes abgeschlossen haben, sollten Sie die Ressourcen, die Sie in dieser Übung erstellt haben, löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zu der Browserregisterkarte zurück, die das Azure-Portal enthält (oder öffnen Sie das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` erneut in einer neuen Browserregisterkarte) und sehen Sie sich den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.

1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.

1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
