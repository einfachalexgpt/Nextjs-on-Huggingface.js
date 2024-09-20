
---
title: "Next.js auf \U0001F917 Spaces"
emoji: "\U0001F433\U0001F917"
colorFrom: blue
colorTo: yellow
sdk: docker
pinned: false
license: agpl-3.0
app_port: 3000
---
<h1 align="center">Next.js auf 🤗 Spaces</h1>

<p align="center">
Führe deine ML-Demo einfach in einer <a href="https://nextjs.org">Next.js</a>-Umgebung aus.
</p>

Bei failfast sind wir begeistert davon, Demos mit TypeScript, Next.js und MUI zu erstellen. Inspiriert von der Benutzerfreundlichkeit von Gradio und Streamlit innerhalb von Hugging Face Spaces möchten wir ein ähnliches Entwicklererlebnis für JavaScript-Enthusiasten bieten. Unser Toolkit enthält vordefinierte MUI-Komponenten, die es dir ermöglichen, intuitive Benutzeroberflächen für deine ML-Demos zu erstellen.

---

<!-- toc -->

- [Lokale Entwicklung](#lokale-entwicklung)
  * [Verwende den Docker-Container lokal](#verwende-den-docker-container-lokal)
- [Verwaltung von Geheimnissen](#verwaltung-von-geheimnissen)
  * [Build-Zeit](#build-zeit)
  * [Laufzeit](#laufzeit)
- [Dockerize ein bestehendes Projekt](#dockerize-ein-bestehendes-projekt)
- [Synchronisiere dein GitHub-Repository mit deinem 🤗 Space](#synchronisiere-dein-github-repository-mit-deinem-%F0%9F%A4%97-space)
- [Bereinige deinen 🤗 Space](#bereinige-deinen-%F0%9F%A4%97-space)
- [Entwicklungs-Roadmap](#entwicklungs-roadmap)

<!-- tocstop -->

---

## Lokale Entwicklung

1. Installiere die Abhängigkeiten: `npm i`
2. Starte den lokalen Entwicklungs-Server: `npm run dev`
3. Öffne die App über [localhost:3000](http://localhost:3000)

### Verwende den Docker-Container lokal

> ℹ️ Damit die Befehle funktionieren, benötigst du mindestens Docker >= 20.10, da wir Umgebungsvariablen als Geheimnisse verwenden.

Um sicherzustellen, dass alles funktioniert, kannst du deinen Container lokal ausführen:

1. [Installiere Docker](https://docs.docker.com/get-docker/) auf deinem Rechner.
2. Gehe in den Ordner `nextjs-hf-spaces`.
3. Erstelle dein Docker-Image: `docker build -t nextjs-hf-spaces .`
4. Starte deinen Docker-Container: `docker run -p 3000:3000 nextjs-hf-spaces`.
5. Öffne die App über [localhost:3000](http://localhost:3000).

Wenn du auch ein Geheimnis hast, das in den Container übergeben werden muss, kannst du dies tun:

1. Erstelle eine Kopie von `.env.local.example` und benenne sie in `.env.local` um (sie enthält das Geheimnis `HF_EXAMPLE_SECRET`).
2. Starte deinen Docker-Container und gib die Umgebungsdatei an: `docker run -p 3000:3000 --env-file .env.local nextjs-hf-spaces`.
3. Öffne die Beispiel-API über [localhost:3000/api/env](http://localhost:3000/api/env) und überprüfe, ob der Wert unseres Geheimnisses `HF_EXAMPLE_SECRET` angezeigt wird.

## Verwaltung von Geheimnissen

Um deine Geheimnisse nicht für Endbenutzer sichtbar zu machen, kannst du sie direkt in den **Einstellungen** deines 🤗 Spaces hinzufügen.

1. Öffne deinen Space und navigiere zu den **Einstellungen**.
2. Finde **Repository Secrets** und klicke auf **Neues Geheimnis**.

Das war's, du kannst nun auf dein Geheimnis zugreifen.

### Build-Zeit

Wenn du während der Build-Zeit ein Geheimnis benötigst (z.B. möchtest du private npm-Pakete installieren), kannst du es direkt in die `Dockerfile` einfügen:

```dockerfile
# Kommentiere die folgenden Zeilen ein, wenn du ein Geheimnis während der Buildzeit verwenden möchtest,
# z.B. um auf deine privaten npm-Pakete zuzugreifen
RUN --mount=type=secret,id=HF_EXAMPLE_SECRET,mode=0444,required=true \
    $(cat /run/secrets/HF_EXAMPLE_SECRET)
```

In diesem Fall mounten wir das Geheimnis `HF_EXAMPLE_SECRET` (unter Verwendung von [Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)) und können es verwenden.

### Laufzeit

Wenn dein 🤗 Space läuft und du ein Geheimnis verwenden möchtest (z.B. um auf eine API zuzugreifen, die eine Authentifizierung erfordert), ohne es dem Benutzer preiszugeben, kannst du es als Umgebungsvariable über `process.env` verwenden.

```typescript
import process from "node:process";
import { NextApiRequest, NextApiResponse } from "next";

export default async function handler(
  request: NextApiRequest,
  response: NextApiResponse
) {
  const exampleSecret = process.env.HF_EXAMPLE_SECRET;

  // Deine Logik, um auf eine API zuzugreifen, die eine Authentifizierung erfordert

  return response.status(200).json("Wir haben Zugriff auf eine externe API");
}
```

Ein einfaches Beispiel findest du unter [nextjs-hf-spaces/api/env](https://huggingface.co/spaces/failfast/nextjs-hf-spaces/api/env). Dies gibt das Geheimnis zurück, um zu sehen, dass es funktioniert, aber du würdest dies in deinem Space nicht tun, da du das Geheimnis nicht einem Endbenutzer zeigen möchtest.

## Dockerize ein bestehendes Projekt

Um Unterstützung für Docker zu einem bestehenden Projekt hinzuzufügen, kopiere einfach die `Dockerfile` in das Stammverzeichnis des Projekts und füge Folgendes in die `next.config.js` Datei ein:

```js
// next.config.js
module.exports = {
  // ... restliche Konfiguration
  output: "standalone",
};
```

Dies baut das Projekt als eigenständige App innerhalb des Docker-Images.

## Synchronisiere dein GitHub-Repository mit deinem 🤗 Space

Wenn du alle Funktionen für kollaborative Entwicklung auf GitHub nutzen möchtest, aber deine Demo auf 🤗 Spaces behalten willst, kannst du eine GitHub-Aktion einrichten, die Änderungen automatisch von GitHub in Spaces überträgt.

> ℹ️ Git-LFS ist erforderlich für Dateien größer als 10 MB.

1. Erstelle dein Repository auf GitHub.
2. Erstelle ein [GitHub-Geheimnis](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) namens `HF_TOKEN` und verwende ein [Zugriffstoken von Hugging Face](https://huggingface.co/settings/tokens) als dessen Wert (du musst eingeloggt sein, um dies zu tun).
3. Aktualisiere den Workflow [sync_to_hf_spaces.yml](.github/workflows/sync_to_hf_spaces.yml).
   - Konfiguriere `HF_USERNAME`: Ersetze `failfast` durch den Namen deines 🤗 Benutzerkontos oder deiner 🤗 Organisation.
   - Konfiguriere `HF_SPACE_NAME`: Ersetze `nextjs-hf-spaces` durch den Namen deines 🤗 Spaces.
4. Push den Code in dein Repository auf GitHub.

Dies sollte Änderungen im **main** Branch von GitHub in deinen 🤗 Space übertragen.

Für weitere Informationen kannst du den [Leitfaden auf Hugging Face](https://huggingface.co/docs/hub/spaces-github-actions) nachlesen.

## Bereinige deinen 🤗 Space

Du benötigst nicht alle Demo-Inhalte und Beispiele? Dann kannst du diese Ressourcen löschen, um einen sauberen 🤗 Space zu erhalten:

* `src/pages/api/env.ts`
* `src/components/example-components.tsx`
* `src/components/getting-started.tsx`
* `src/components/under-construction.tsx`
* `src/components/title.tsx`
* `src/components/huggingface/huggingface.tsx`

Aktualisiere die Datei `src/components/index.tsx` und entferne:

```jsx
<Title />

<GettingStarted />

<DividerBox />

<ExampleComponents />
```

> Hast du eine Idee, wie das besser gemacht werden könnte? Lass es uns wissen!

## Entwicklungs-Roadmap

Die nächsten Meilensteine in beliebiger Reihenfolge sind:

* Komponenten für alle [`@huggingface/inference`](https://huggingface.co/docs/huggingface.js/inference/README) Methoden (in Arbeit)
* Komponenten zur Nutzung von [langchain.js](https://js.langchain.com/docs)
* Komponenten zur Nutzung von [hyv](https://github.com/failfa-st/hyv)
* Veröffentliche Komponenten auf npm, um sie außerhalb von [nextjs-hf-spaces](https://github.com/failfa-st/nextjs-hf-spaces) nutzbar zu machen.
* Bereitstellung von Vorlagen für verschiedene Anwendungsfälle, die zu komplex für einzelne Komponenten sind.
* Dokumentation zur Verwendung der Komponenten mit allen verfügbaren Optionen.

> Fehlt etwas? Lass es uns wissen!
```

