---
lab:
  title: Entwickeln eines KI-Agents mit VS Code-Erweiterung
  description: 'Verwenden Sie die VS Code-Erweiterung von Microsoft Foundry, um einen KI-Agent zu erstellen.'
---

# Entwickeln eines KI-Agents mit VS Code-Erweiterung

In dieser Übung verwenden Sie die VS Code-Erweiterung von Microsoft Foundry, um einen Agent zu erstellen, der MCP-Servertools (Model Context Protocol) verwenden kann, um auf externe Datenquellen und APIs zuzugreifen. Der Agent kann aktuelle Informationen abrufen und über MCP-Tools mit verschiedenen Diensten interagieren.

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Einige der in dieser Übung verwendeten Technologien befinden sich in der Vorschau oder in der aktiven Entwicklung. Es kann zu unerwartetem Verhalten, Warnungen oder Fehlern kommen.

## Voraussetzungen

Bevor Sie mit dieser Übung beginnen, stellen Sie sicher, dass Sie Folgendes erfüllt haben:
- Installierte Visual Studio Code-Instanz
- Ein aktives Azure-Abonnement

## Installieren der VS Code-Erweiterung von Foundry

Beginnen wir mit der Installation und Einrichtung der VS Code-Erweiterung.

1. Öffnen Sie Visual Studio Code.

1. Wählen Sie im linken Bereich **Erweiterungen** aus (oder drücken Sie **STRG+Umschalttaste+X**).

1. Geben Sie in der Suchleiste **Foundry** ein und drücken Sie die Eingabetaste.

1. Wählen Sie die **Foundry**-Erweiterung von Microsoft aus und klicken Sie auf **Installieren**.

1. Überprüfen Sie nach Abschluss der Installation, ob die Erweiterung in der primären Navigationsleiste auf der linken Seite von Visual Studio Code angezeigt wird.

## Anmelden bei Azure und Erstellen eines Projekts

Stellen Sie jetzt eine Verbindung mit Ihren Azure-Ressourcen her und erstellen ein neues AI Foundry-Projekt.

1. Wählen Sie in der VS Code-Randleiste das Symbol für die **Foundry**-Erweiterung aus.

1. Wählen Sie in der Azure-Ressourcenansicht die Option **Bei Azure anmelden …** aus und folgen Sie den Authentifizierungsanweisungen.

1. Wählen Sie nach der Anmeldung Ihr Azure-Abonnement aus dem Dropdownmenü aus.

1. Erstellen Sie ein neues Foundry-Projekt, indem Sie das Symbol **+** (Plus) neben **Ressourcen** in der Foundry-Erweiterungsansicht auswählen.

1. Wählen Sie aus, ob sie eine neue Ressourcengruppe erstellen oder eine vorhandene verwenden möchten:
   
   **Erstellen einer neuen Ressourcengruppe:**
   - Wählen Sie **Neue Ressourcengruppe erstellen** aus und drücken Sie die Eingabetaste
   - Geben Sie einen Namen für Ihre Ressourcengruppe ein (z. B. „rg-ai-agents-lab“) und drücken Sie die Eingabetaste
   - Wählen Sie einen Speicherort aus den verfügbaren Optionen aus und drücken Sie die Eingabetaste
   
   **Verwenden einer vorhandenen Ressourcengruppe:**
   - Wählen Sie die zu verwendende Ressourcengruppe aus der Liste aus und drücken Sie die Eingabetaste

1. Geben Sie im Textfeld einen Namen für Ihr Foundry-Projekt (z. B. „ai-agents-project“) ein und drücken Sie die Eingabetaste.

1. Warten Sie, bis die Projektbereitstellung abgeschlossen ist. Ein Popup mit der Meldung „Projekt erfolgreich bereitgestellt“ wird angezeigt.

## Bereitstellen eines Modells

Sie benötigen ein bereitgestelltes Modell, das mit Ihrem Agent verwendet wird.

1. Wenn das Popup „Projekt erfolgreich bereitgestellt“ angezeigt wird, wählen Sie die Schaltfläche **Modell bereitstellen** aus. Dadurch wird der Modellkatalog geöffnet.

   > **Tipp**: Sie können auch auf den Modellkatalog zugreifen, indem Sie das Symbol **+** neben **Modelle** im Abschnitt „Ressourcen“ auswählen oder **F1** drücken und den Befehl **Foundry: Open Model Catalog** ausführen.

1. Suchen Sie im Modellkatalog nach dem Modell **gpt-4o** (Sie können die Suchleiste verwenden, um es schnell zu finden).

    ![Screenshot des Modellkatalogs in der VS Code-Erweiterung von Foundry.](Media/vs-code-model.png)

1. Wählen Sie **In Azure bereitstellen** neben dem gpt-4o-Modell aus.

1. Konfigurieren Sie die Bereitstellungseinstellungen:
   - **Bereitstellungsname:** Geben Sie einen Namen wie „gpt-4o-deployment“ ein
   - **Bereitstellungstyp**: Wählen Sie **Globaler Standard** aus (oder **Standard**, wenn „Globaler Standard“ nicht verfügbar ist)
   - **Modellversion**: Als Standard beibehalten
   - **Token pro Minute**: Als Standard beibehalten

1. Wählen Sie **In Foundry bereitstellen** in der unteren linken Ecke aus.

1. Wählen Sie im Bestätigungsdialogfeld **Bereitstellen** aus, um das Modell bereitzustellen.

1. Warten Sie, bis die Bereitstellung abgeschlossen ist. Ihr bereitgestelltes Modell wird im Abschnitt **Modelle** der Ressourcenansicht angezeigt.

## Erstellen eines KI-Agents mit der Ansicht des Designers

Jetzt erstellen Sie einen KI-Agent mithilfe der Oberfläche des Visual Designers.

1. Suchen Sie in der Foundry-Erweiterungsansicht den Abschnitt **Ressourcen**.

1. Wählen Sie das Symbol **+** (Plus) neben dem Unterabschnitt **Agents** aus, um einen neuen KI-Agent zu erstellen.

    ![Screenshot eines Agents in der VS Code-Erweiterung von Foundry.](Media/vs-code-new-agent.png)

1. Wählen Sie einen Speicherort für Ihre Agentdateien aus, wenn Sie dazu aufgefordert werden.

1. Die Ansicht des Agent-Designers wird zusammen mit einer `.yaml`-Konfigurationsdatei geöffnet.

### Konfigurieren Ihres Agents im Designer

1. Konfigurieren Sie im Agent-Designer die folgenden Felder:
   - **Name**: Geben Sie einen beschreibenden Namen für Ihren Agent ein (z. B. „data-research-agent“)
   - **Beschreibung:** Fügen Sie eine Beschreibung des Zwecks des Agents hinzu
   - **Modell**: Wählen Sie Ihre GPT-4o-Bereitstellung aus dem Dropdownmenü aus
   - **Anleitung:** Geben Sie Systemanweisungen ein, z. B.:
     ```
     You are an AI agent that helps users research information from various sources. Use the available tools to access up-to-date information and provide comprehensive responses based on external data sources.
     ```

1. Speichern Sie die Konfiguration, indem Sie **Datei > Speichern** in der Menüleiste „VS-Code“ auswählen.

## Hinzufügen eines MCP-Servertools zu Ihrem Agent

Sie fügen nun ein MCP-Servertool (Model Context Protocol) hinzu, mit dem Ihr Agent auf externe APIs und Datenquellen zugreifen kann.

1. Wählen Sie im Abschnitt **TOOL** des Designers die Schaltfläche **Tool hinzufügen** in der oberen rechten Ecke aus.

![Screenshot des Hinzufügens eines Tools zu einem Agent in der VS Code-Erweiterung von Foundry.](Media/vs-code-agent-tools.png)

1. Wählen Sie im Dropdownmenü **MCP-Server** aus.

1. Konfigurieren Sie das MCP-Servertool mit den folgenden Informationen:
   - **Server-URL**: Geben Sie die URL eines MCP-Servers ein (z. B. `https://gitmcp.io/Azure/azure-rest-api-specs`)
   - **Serverbezeichnung**: Geben Sie einen eindeutigen Bezeichner ein (z. B. „github_docs_server“)

1. Lassen Sie das Dropdownmenü **Zulässige Tools** leer, um alle Tools vom MCP-Server zuzulassen.

1. Wählen Sie die Schaltfläche **Tool erstellen** aus, um das Tool Ihrem Agent hinzuzufügen.

## Bereitstellen Ihres Agents in Foundry

1. Wählen Sie in der Ansicht des Designers in der unteren linken Ecke die Schaltfläche **Auf Foundry erstellen** aus.

1. Warten Sie, bis die Bereitstellung abgeschlossen ist.

1. Aktualisieren Sie in der Navigationsleiste von VS Code die Ansicht **Azure-Ressourcen**. Ihr bereitgestellter Agent sollte nun unter dem Unterabschnitt **Agents** angezeigt werden.

## Testen des Agents im Playground

1. Klicken Sie im Unterabschnitt **Agents ** mit der rechten Maustaste auf Ihren bereitgestellten Agent.

1. Wählen Sie im Kontextmenü **Playground öffnen** aus.

1. Der Agents-Playground wird in VS Code auf einer neuen Registerkarte geöffnet.

1. Geben Sie eine Test-Prompt ein, z. B.:

   ```output
   Can you help me find documentation about Azure Container Apps and provide an example of how to create one?
   ```

1. Senden Sie die Nachricht und beobachten Sie die Authentifizierungs- und Genehmigungs-Prompts für das MCP-Servertool:
    - Wählen Sie für diese Übung **Keine Authentifizierung** aus, wenn Sie dazu aufgefordert werden.
    - Für die Genehmigungseinstellung der MCP-Tools können Sie **Immer genehmigen** auswählen.

1. Überprüfen Sie die Antwort des Agents und beachten Sie, wie das MCP-Servertool zum Abrufen externer Informationen verwendet wird.

1. Überprüfen Sie den Abschnitt **Agentanmerkungen**, um die vom Agent verwendeten Informationsquellen anzuzeigen.

## Generieren von Beispielcode für Ihren Agent

1. Klicken Sie mit der rechten Maustaste auf Ihren bereitgestellten Agent und wählen Sie **Codedatei öffnen** aus, oder wählen Sie auf der Seite „Agenteinstellungen“ die Schaltfläche **Codedatei öffnen** aus.

1. Wählen Sie Ihr bevorzugtes SDK aus dem Dropdownmenü (Python, .NET, JavaScript oder Java) aus.

1. Wählen Sie Ihre bevorzugte Programmiersprache aus.

1. Wählen Sie Ihre bevorzugte Authentifizierungsmethode.

1. Überprüfen Sie den generierten Beispielcode, der die programmgesteuerte Interaktion mit Ihrem Agent veranschaulicht.

Sie können diesen Code als Ausgangspunkt für die Erstellung von Anwendungen verwenden, die Ihren KI-Agent nutzen.

## Anzeigen von Unterhaltungsverlauf und Threads

1. Erweitern Sie in der Ansicht **Azure-Ressourcen** den Unterabschnitt **Threads**, um Unterhaltungen anzuzeigen, die während Ihrer Agentinteraktionen erstellt wurden.

1. Wählen Sie einen Thread aus, um die Seite **Threaddetails** mit Folgendem anzuzeigen:
   - Einzelne Nachrichten in der Unterhaltung
   - Ausführungsinformationen und -details
   - Agentantworten und Toolverwendung

1. Wählen Sie **Ausführungsinformationen anzeigen** aus, um detaillierte JSON-Informationen zu jeder Ausführung anzuzeigen.

## Zusammenfassung

In dieser Übung haben Sie die VS Code-Erweiterung von Foundry verwendet, um einen KI-Agent mit MCP-Servertools zu erstellen. Der Agent kann über das Model Context Protocol auf externe Datenquellen und APIs zugreifen und so aktuelle Informationen bereitstellen und mit verschiedenen Diensten interagieren. Außerdem haben Sie gelernt, wie Sie den Agent im Playground testen und Beispielcode für die programmgesteuerte Interaktion generieren.

## Bereinigen

Wenn Sie mit der Erkundung der VS Code-Erweiterung von Foundry fertig sind, sollten Sie die Ressourcen bereinigen, um unnötige Azure-Kosten zu vermeiden.

### Löschen Ihrer Agents

1. Wählen Sie im Navigationsmenü des Foundry-Portals **Agents** aus.

1. Wählen Sie Ihren Agent und dann die Schaltfläche **Löschen** aus.

### Löschen Ihrer Modelle

1. Aktualisieren Sie in VS Code die Ansicht **Azure-Ressourcen**.

1. Erweitern Sie den Unterabschnitt **Modelle**.

1. Klicken Sie mit der rechten Maustaste auf Ihr bereitgestelltes Modell und wählen Sie **Löschen** aus.

### Löschen anderer Ressourcen

1. Öffnen Sie das [Azure-Portal](https://portal.azure.com).

1. Navigieren Sie zu der Ressourcengruppe, die Ihre AI Foundry-Ressourcen enthält.

1. Wählen Sie **Ressourcengruppe löschen** aus und bestätigen Sie den Löschvorgang.