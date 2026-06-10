# ❌️ Defender Remover / Defender Disabler

<a href="https://github.com/murdok1982/windows-defender-remover">
    <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://github.com/drunkwinter/windows-defender-remover/assets/38593134/8072a566-5bf0-4f05-9994-808145406bdc">
        <img alt="Defender Remover" src="https://user-images.githubusercontent.com/79479952/239704528-c017473e-1d2a-4d4a-a215-bf71d137b86a.png">
    </picture>
</a>

## ❓️ ¿Qué hace esta aplicación? / What does the app do?

Esta aplicación remueve/deshabilita Windows Defender, incluyendo Windows Security App, Windows Virtualization-Based Security (VBS), Windows SmartScreen, Windows Security Services, Windows Web-Threat Service, Windows File Virtualization (UAC), Microsoft Defender App Guard, Microsoft Driver Block List, System Mitigations y la página de Windows Defender en la aplicación de Configuración en Windows 10 o posterior.

This application removes/disables Windows Defender, including the Windows Security App, Windows Virtualization-Based Security (VBS), Windows SmartScreen, Windows Security Services, Windows Web-Threat Service, Windows File Virtualization (UAC), Microsoft Defender App Guard, Microsoft Driver Block List, System Mitigations and the Windows Defender page in the Settings App on Windows 10 or later.

## ❓️ ¿Qué componentes se eliminan? / What components are removed?

### Removing Security Components
    This script removes/disables following security components:
        - Support for Windows Security Center including Windows Security Center Service (wscsvc), Windows Security Service (SgrmBroker, Sgrm Drivers) which are needed to run Windows Security App.
        - Virtualization support:
            - Hypervisor startup (this fixes disabling of Virtualization Based Security, this will auto enable if you use Hyper-V and/or WSL (Windows Subsystem for Linux), WSA (Windows Subsystem for Android))
            - LUA (disables File Virtualization and User Account Control, which will run all apps as administrator privileges (also fixes old app errors))
            - Exploit Guard
            - Windows Smart Control
            - Tamper Protection (for Windows 11 21H2 or earlier)
        - SecHealthUI (Windows Security UWP App)
        - SmartScreen
        - Pluton Support and Pluton Services Support
        - System Mitigations:
          - "Services Mitigations" (search on admx.help for more information)
          - Spectre and Meltdown Mitigation (for +30% performance on old Intel CPUs)
        - Windows Security Section from Settings App

### Removing Antivirus Components
    This script forcibly removes following antivirus components:
      - Windows Defender Definition Update List (this will disable updating definitions of Defender because it's removed)
      - Windows Defender SpyNet Telemetry
      - Antivirus Service
      - Windows Defender Antivirus filter and windows defender rootkit scanner drivers
      - Antivirus Scanning Tasks
      - Shell Associations (Context Menu)
      - Hides Antivirus Protection section from Windows Security App

## 📃 Instrucciones / Instructions

> [!NOTE]
> Se recomienda crear un punto de restauración del sistema antes de ejecutar el script. / A system restore point is recommended before you run the script (if you don't know what you are doing).

> [!WARNING]
> Este script modifica componentes críticos de seguridad de Windows. Úsalo bajo tu propia responsabilidad. / This script modifies critical Windows security components. Use at your own risk.

1. Descarga el script empaquetado desde [Releases](https://github.com/murdok1982/windows-defender-remover/releases) / Download the packed script from Releases
2. Ejecuta el ".exe" como administrador / Run the ".exe" as administrator
3. Sigue las instrucciones mostradas / Follow the instructions displayed

**OR** you can use git:

```bash
git clone https://github.com/murdok1982/windows-defender-remover.git
cd windows-defender-remover
Script_Run.bat
```

**OR** you can download entire source code:

1. Download the source code from [Releases](https://github.com/murdok1982/windows-defender-remover/releases)
2. Choose the file **Source Code(.zip)** from last version and download it
3. Unarchive the file into a folder and run the Script_Run.bat

You can file an [issue](https://github.com/murdok1982/windows-defender-remover/issues) if you experience any problems.

## 📃 Automatización del script / Script Automation

Puedes remover Defender con argumentos. / You can remove Defender with arguments.

#### Removing

```PowerShell
# Removal
Defender.Remover.exe /r <# or /R #>
```

## Disable or Remove Windows Defender Application Guard Policies (advanced)

If you have any problems when opening an app (*extremely rare*) and get the message "The app can not run because Device Guard" or "Windows Defender Application Guard Blocked this app", you have to remove 4 files with the same name, from different locations.

- In EFI Partition

```PowerShell
Remove-Item -LiteralPath "$((Get-Partition | ? IsSystem).AccessPaths[0])Microsoft\Boot\WiSiPolicy.p7b"
```

- In Code Integrity Folder

```PowerShell
Remove-Item -LiteralPath "$env:windir\System32\CodeIntegrity\WiSiPolicy.p7b"
```

- In Windows Folder

```PowerShell
Remove-Item -LiteralPath "$env:windir\Boot\EFI\wisipolicy.p7b"
```

- In WinSxS Folder

```PowerShell
Remove-Item -Path "$env:windir\WinSxS" -Include *winsipolicy.p7b* -Recurse
```

## Creating an ISO with Windows Defender and Services disabled

You can create an ISO with Windows Defender and Security Services Disabled. It's easy, so this is a file which can help you.

Here are the steps:

1. Mount the ISO and extract it into a location
2. Open the **sources** folder and create the **$OEM$** folder (this is needed to run the DefenderRemover part in OOBE)
3. Open the **$OEM$** folder and create the folder with **$$** name
4. Open the **$$** folder and create the folder with **Panther** name
5. Open the **Panther** folder. The path should look like: **%location of extracted ISO%\sources\$OEM$\$$\Panther\**
6. Download the unattended.xml file from repo in ISO_Maker folder and put it in Panther folder
7. Save this as bootable ISO (for now the script can't do this automatically, but it will in next version)

## ❓ Preguntas Frecuentes / Frequently Asked Questions

#### ⭕ How to remove Windows Security Center / Windows Security App from PC without downloading Script?

Paste this code into a PowerShell file and **Run as Administrator**:

```PowerShell
$remove_appx = @("SecHealthUI"); $provisioned = get-appxprovisionedpackage -online; $appxpackage = get-appxpackage -allusers; $eol = @()
$store = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore'
$users = @('S-1-5-18'); if (test-path $store) {$users += $((dir $store -ea 0 |where {$_ -like '*S-1-5-21*'}).PSChildName)}
foreach ($choice in $remove_appx) { if ('' -eq $choice.Trim()) {continue}
  foreach ($appx in $($provisioned |where {$_.PackageName -like "*$choice*"})) {
    $next = !1; foreach ($no in $skip) {if ($appx.PackageName -like "*$no*") {$next = !0}} ; if ($next) {continue}
    $PackageName = $appx.PackageName; $PackageFamilyName = ($appxpackage |where {$_.Name -eq $appx.DisplayName}).PackageFamilyName 
    ni "$store\Deprovisioned\$PackageFamilyName" -force >''; $PackageFamilyName  
    foreach ($sid in $users) {ni "$store\EndOfLife\$sid\$PackageName" -force >''} ; $eol += $PackageName
    dism /online /set-nonremovableapppolicy /packagefamily:$PackageFamilyName /nonremovable:0 >''
    remove-appxprovisionedpackage -packagename $PackageName -online -allusers >''
  }
  foreach ($appx in $($appxpackage |where {$_.PackageFullName -like "*$choice*"})) {
    $next = !1; foreach ($no in $skip) {if ($appx.PackageFullName -like "*$no*") {$next = !0}} ; if ($next) {continue}
    $PackageFullName = $appx.PackageFullName; 
    ni "$store\Deprovisioned\$appx.PackageFamilyName" -force >''; $PackageFullName
    foreach ($sid in $users) {ni "$store\EndOfLife\$sid\$PackageFullName" -force >''} ; $eol += $PackageFullName
    dism /online /set-nonremovableapppolicy /packagefamily:$PackageFamilyName /nonremovable:0 >''
    remove-appxpackage -package $PackageFullName -allusers >''
  }
}
```

#### ⭕ Why is the downloaded executable being flagged as a virus?

That is a **false positive**.

Some security apps flag this app as a virus because of the way the ".exe" files are created. Download with **git** or source code .zip will indicate virus-free.

#### ⭕ Why is the patch not working when Windows is updated?

Windows Update includes an ```Intelligence Update``` which blocks certain actions and modifies Windows Defender/Security policies.
If the script is not working for you, check if you have the Windows Security Intelligence Update installed. If you do, disable tamper protection, and re-run the script.

#### ⭕ How to use the package remover without downloading the executable from the release?

Run the desired ".bat" file from cmd with PowerRun (by dragging to the executable). You must reboot for the changes to take effect.

#### ⭕ How to disable VBS if the removal script does not work

Disable with this command and reboot:

```bash
bcdedit /set hypervisorlaunchtype off
```

After that you will not be able to use virtual machines.

#### ⭕ Why VBS keeps enabling on Windows 11?

By default the script is disabling VBS to gain performance in your system. The factors which keep VBS enabled are related to Windows Virtualization.

Apps and features which use Windows Virtualization:

- Windows Subsystem for **Android**/**Linux** (WSA/WSL)
- Hyper-V Virtual Machine
- Microsoft Emulator (Windows 10X Emulator which you can find in Microsoft Store)
- Android Studio integration in Visual Studio or other Emulators (for Windows 10 22H2 with March 2025 Update or newer)

If you open any of those apps mentioned earlier, VBS will be enabled without user intervention. It's needed to run Virtual Machine engine. If you don't use any virtual machine, you can file an [Issue here](https://github.com/murdok1982/windows-defender-remover/issues).

---

## 💰 Donaciones / Donations

Si este proyecto te ha sido útil y quieres apoyar su desarrollo, puedes hacer una donación en Bitcoin:

**If this project has been useful and you want to support its development, you can make a Bitcoin donation:**

### Bitcoin (BTC)
```
bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh
```

¡Cualquier contribución es muy apreciada! 🙏

**Any contribution is greatly appreciated!** 🙏

---

## 👥 Autoría y Créditos / Authorship and Credits

### 🔧 Autor Original / Original Author

**Ionut Bara** - [ionuttbara](https://github.com/ionuttbara)
- Repositorio original / Original repository: [ionuttbara/windows-defender-remover](https://github.com/ionuttbara/windows-defender-remover)
- Creador del proyecto base / Creator of the base project

### ✨ Correcciones y Mejoras / Corrections and Improvements

**Gustavo Lobato Clara (Murdok)** - [murdok1982](https://github.com/murdok1982)

**Mejoras realizadas en esta versión / Improvements made in this version:**
- ✅ Corrección de errores tipográficos en la documentación
- ✅ Mejora en la estructura y formato del README
- ✅ Añadida sección de donaciones Bitcoin
- ✅ Documentación bilingüe (Español/Inglés)
- ✅ Mejoras en advertencias de seguridad
- ✅ Corrección de enlaces rotos
- ✅ Mejor organización de las secciones

**Contacto / Contact:**
- 📧 Email: gustavolobatoclara@gmail.com
- 💼 LinkedIn: [Gustavo Lobato Clara](https://www.linkedin.com/in/gustavo-lobato-clara1982/)
- 🌍 Location: Valencia, España
- 💻 *Apasionado por la ciber-seguridad y PYTHON!!!*

---

## 📝 Licencia / License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## ⚠️ Disclaimer / Descargo de responsabilidad

**ES:** Este software se proporciona "tal cual", sin garantías de ningún tipo. Los autores no se hacen responsables de ningún daño causado por el uso de este software. Usar bajo tu propia responsabilidad.

**EN:** This software is provided "as is", without warranty of any kind. The authors are not responsible for any damage caused by the use of this software. Use at your own risk.
---

## 🎖️ CENTRO DE COMUNICACIONES Y REPORTES OFICIALES
**NIVEL DE ACCESO:** AUTORIZADO | **DESTINATARIO:** COMANDANCIA DE DESARROLLO (gustavolobatoclara@gmail.com)

A través del siguiente portal de comunicaciones, el personal autorizado puede emitir reportes de incidencias, fallas críticas en despliegue (compilación) o solicitudes de mejoras estratégicas. Seleccione la directiva correspondiente para visualizar los protocolos de envío:

<details>
<summary><b>🚨 REPORTAR QUEJA O INCIDENCIA DISCIPLINARIA / OPERATIVA</b></summary>
<br>
Para tramitar una queja sobre el funcionamiento, estructura o contenido del sistema, envíe un mensaje a <b>gustavolobatoclara@gmail.com</b> siguiendo este protocolo:
<ol>
  <li><b>Asunto:</b> [QUEJA] - Nombre del Sistema - Breve descripción.</li>
  <li><b>Cuerpo del mensaje:</b> Detallar claramente la incidencia, impacto operativo y, si es posible, la evidencia (capturas o logs).</li>
  <li><b>Prioridad:</b> Indicar si es de atención inmediata o diferida.</li>
</ol>
</details>

<details>
<summary><b>🛠️ REPORTE DE PROBLEMAS DE COMPILACIÓN O DESPLIEGUE</b></summary>
<br>
Si experimenta fallos durante la fase de compilación o instalación del sistema, reporte a <b>gustavolobatoclara@gmail.com</b> con la siguiente estructura técnica:
<ol>
  <li><b>Asunto:</b> [COMPILACIÓN] - Falla en entorno &lt;Entorno/OS&gt;.</li>
  <li><b>Especificaciones:</b> Sistema Operativo, versión de dependencias y herramientas de compilación utilizadas.</li>
  <li><b>Traza de Error (Logs):</b> Adjunte el log completo de errores proporcionado por la terminal (en formato texto o captura legible).</li>
  <li><b>Pasos de Reproducción:</b> Secuencia exacta de comandos ejecutados antes del fallo crítico.</li>
</ol>
</details>

<details>
<summary><b>💡 SUGERENCIAS O SOLICITUDES DE DESARROLLO</b></summary>
<br>
Para proponer nuevas capacidades tácticas, módulos de inteligencia o mejoras de arquitectura, envíe su solicitud a <b>gustavolobatoclara@gmail.com</b>:
<ol>
  <li><b>Asunto:</b> [PROPUESTA] - Mejora o Nuevo Módulo.</li>
  <li><b>Objetivo Táctico:</b> ¿Qué problema resuelve o qué ventaja proporciona esta nueva característica?</li>
  <li><b>Viabilidad:</b> (Opcional) Posible enfoque técnico o herramientas recomendadas para su implementación.</li>
</ol>
</details>

---

---

## Support / Apoya este proyecto

I build open-source projects focused on applied AI, automation, and data intelligence.
Over on my GitHub you'll find things like AI-powered analysis engines, OSINT platforms for open-source research, Windows automation tools, and experiments with language models.
Everything is public and free, so anyone can use it, study it, or build on top of it. github.com/murdok1982

Keeping these projects alive takes a lot of hours. If any of them have helped you out or you just like what I'm doing, you can support me with a coffee: ko-fi.com/murdok1982

Every contribution goes straight back into shipping more open-source code.
