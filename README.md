# grimot-launcher-install — Publicação (GitHub Pages)

Este repositório serve os ficheiros necessários para o instalador e o launcher do **GrimOT** via **GitHub Pages**.

Site: `https://grimotz.github.io/grimot-launcher-install/`

---

## Estrutura

```
grimot-launcher-install/
├── docs/
│   └── launcher_config.json        ← configuração do cliente (versão, URL do ZIP, etc.)
└── launcher_updater/
    ├── launchermetadata.json        ← o instalador lê daqui o URL do package.json
    └── launcher/
        ├── package.json             ← lista de ficheiros do launcher e URLs de download
        ├── package.json.version     ← hash/versão do package
        └── download/
            ├── GrimOT.exe
            ├── GrimOT.dll
            └── ...                  ← todos os outros ficheiros do build do launcher
```

---

## Ficheiros

### `docs/launcher_config.json`

Lido pelo **launcher** (já instalado) para obter:
- `clientVersion` – versão atual do cliente do jogo.
- `newClientUrl` – URL do ZIP do cliente (ex.: Google Cloud Storage). O download é feito **direto para essa URL**, não passa pelo GitHub.
- `clientFolder`, `clientExecutable`, `replaceFolders`, etc.

Exemplo:

```json
{
    "clientVersion": "15.03.020",
    "launcherVersion": "2.0.0",
    "replaceFolders": true,
    "replaceFolderName": [
        { "name": "assets" },
        { "name": "storeimages" },
        { "name": "bin" }
    ],
    "clientFolder": "Grim-OT-main",
    "newClientUrl": "https://storage.googleapis.com/grimot/Grim-OT-main.zip",
    "clientExecutable": "client.exe",
    "clientName": "GrimOT"
}
```

### `launcher_updater/`

Usado pelo **instalador** para descarregar e instalar o launcher em `%LocalAppData%\GrimOTLauncher`.

| Ficheiro | Descrição |
|---------|-----------|
| `launchermetadata.json` | URL do `package.json`; é o primeiro ficheiro que o instalador pede. |
| `launcher/package.json` | Lista de todos os ficheiros do launcher com URLs de download e hashes. |
| `launcher/package.json.version` | Hash do package (usado para detetar atualizações do launcher). |
| `launcher/download/*` | Binários do launcher: `GrimOT.exe`, `GrimOT.dll`, DLLs e recursos. |

---

## Como atualizar (quando há uma nova versão do launcher)

No projeto de código-fonte (`c:\dev\projects\grimot-launcher`):

```powershell
# 1. Compilar o launcher
dotnet build GrimOT.csproj -c Release

# 2. Gerar o manifest e copiar para publish\launcher_updater
.\scripts\GeneratePackageManifest.ps1 `
  -UrlBase "https://grimotz.github.io/grimot-launcher-install/launcher_updater/launcher" `
  -ReleasePath "bin\Release\net6.0-windows" `
  -CopyToPublish

# 3. Copiar para este repositório
Copy-Item -Recurse -Force `
  "c:\dev\projects\grimot-launcher\publish\launcher_updater\*" `
  "c:\dev\projects\grimot-launcher-install\launcher_updater\"
```

Depois, neste repositório:

```powershell
cd c:\dev\projects\grimot-launcher-install
git add .
git commit -m "chore: atualizar launcher vX.X.X"
git push
```

O GitHub Pages publica automaticamente. Os jogadores que abrirem o launcher existente irão receber a nova versão na próxima instalação (o instalador verifica `launchermetadata.json`).

---

## Como atualizar o cliente do jogo

Edita `docs/launcher_config.json` e incrementa o campo `clientVersion`. O launcher compara esta versão com a versão local; se for diferente, mostra o botão de Download.

```powershell
# Após editar docs/launcher_config.json:
cd c:\dev\projects\grimot-launcher-install
git add docs/launcher_config.json
git commit -m "chore: bump clientVersion para X.X.X"
git push
```

---

## Configuração do GitHub Pages

1. **Settings → Pages** neste repositório (`Grimotz/grimot-launcher-install`).
2. Source: **Deploy from a branch**, branch **`main`**, pasta **`/ (root)`**.
3. Guardar. O site fica disponível em `https://grimotz.github.io/grimot-launcher-install/`.

Confirma que os ficheiros estão acessíveis:
- `https://grimotz.github.io/grimot-launcher-install/docs/launcher_config.json`
- `https://grimotz.github.io/grimot-launcher-install/launcher_updater/launchermetadata.json`
- `https://grimotz.github.io/grimot-launcher-install/launcher_updater/launcher/package.json`
- `https://grimotz.github.io/grimot-launcher-install/launcher_updater/launcher/download/GrimOT.exe`
