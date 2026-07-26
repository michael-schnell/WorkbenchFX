# WorkbenchFX Demos

Runnable JavaFX applications showcasing WorkbenchFX. Each demo is wired up as an
`exec-maven-plugin` execution in this module's `pom.xml`, so you can start any of them
straight from Maven.

## Prerequisites

* JDK 21 (the parent POM sets `java.version` to 21)
* A desktop/display — these are GUI applications and will not start headless
* No separate JavaFX SDK needed; the OpenJFX artifacts come in as Maven dependencies

## Running a demo

All commands are run from the **repository root** using the Maven wrapper.

On a fresh clone — and whenever you change `workbenchfx-core` — build and install the whole
project first, then launch the demo:

```bash
./mvnw install -DskipTests && ./mvnw -pl workbenchfx-demo exec:java@custom-demo
```

Once `workbenchfx-core` is in your local repository, changes confined to this module only need:

```bash
./mvnw -pl workbenchfx-demo compile exec:java@custom-demo
```

> **Note:** the `install` step is required because the demo is launched in a separate Maven
> invocation whose reactor contains only this module, so `workbenchfx-core` is resolved as an
> external artifact from `~/.m2`. A `verify` build stops at `target/` and publishes nothing there.
> The current version `21.0.0` is not on Maven Central yet, so skipping `install` fails outright
> with `com.dlsc.workbenchfx:workbenchfx-core:jar:21.0.0 (absent)` rather than silently using a
> stale jar.

> **Note:** do not add `-am`. Maven applies a command-line goal to every module in the reactor
> and falls back to the plugin's default configuration where the execution id is missing, so
> the build fails on `workbenchfx-core` with `The parameters 'mainClass' ... are missing or invalid`.

## Available demos

Replace the execution id in the commands above with any of the following:

| Execution id         | Main class          | What it shows                                                                                 |
|----------------------|---------------------|-----------------------------------------------------------------------------------------------|
| `simple-demo`        | `SimpleDemo`        | Minimal setup: a `Workbench` with a handful of modules (calendar, hello world, maps, web).      |
| `extended-demo`      | `ExtendedDemo`      | Adds toolbars, menus, drawers, dialogs and a PreferencesFX-backed settings module.              |
| `custom-demo`        | `CustomDemo`        | Full customisation — custom page, tab, tile factories, navigation drawer and overlays.           |
| `fxml-demo`          | `FXMLDemo`          | Declaring the `Workbench` in FXML (`workbench.fxml` + `FXMLController`), e.g. with [Scene Builder](https://gluonhq.com/products/scene-builder/). |
| `single-module-demo` | `SingleModuleDemo`  | A `Workbench` with a single module, which hides the tab bar.                                     |

For example, to run the FXML demo:

```bash
./mvnw -pl workbenchfx-demo compile exec:java@fxml-demo
```

## Running from an IDE

Each demo class has a `main` method, so you can also just run it directly from your IDE
after importing the root `pom.xml` as a Maven project.

## Known issues

### WebView modules that open a WebSocket fail on JavaFX 21.0.3 and later

Opening the *JFX-Central* module (`SimpleDemo`, `ExtendedDemo`, `CustomDemo`) throws:

```
java.lang.UnsatisfiedLinkError: 'void com.sun.webkit.network.SocketStreamHandle.twkDidOpen(long)'
```

`https://jfx-central.com` is a Vaadin application and therefore opens a WebSocket, which is
what triggers this. Any WebView page using WebSockets is affected.

This is an upstream JavaFX problem, not a WorkbenchFX one. From JavaFX 21.0.3 onwards the
bundled `libjfxwebkit.so` no longer implements the `SocketStreamHandle` native methods, while
`com.sun.webkit.network.SocketStreamHandle` still declares and calls them. Counting the
`Java_com_sun_webkit_network_SocketStreamHandle_twk*` symbols exported by the native library:

| `javafx-web` | Exported symbols | WebView WebSockets |
|--------------|------------------|--------------------|
| 17.0.1       | 4                | work               |
| 21.0.1       | 4                | work               |
| 21.0.2       | 4                | work — last good release in the 21 line |
| 21.0.3 … 21.0.12 | 0            | broken             |
| 26.0.2       | 0                | broken             |

Since the problem is still present in the newest JavaFX line, upgrading does not help. If you
need WebSockets in a WebView, build against JavaFX 21.0.2:

```bash
./mvnw -pl workbenchfx-demo compile exec:java@simple-demo -Djavafx.version=21.0.2
```

Note that 21.0.2 predates the WebKit fixes shipped in later patch releases, so this is a
workaround for local experimentation rather than a recommendation for production.

## Source layout

```
src/main/java/com/dlsc/workbenchfx/demo/
├── *Demo.java     — the launchable applications listed above
├── controls/      — custom controls and skins used by CustomDemo
└── modules/       — the WorkbenchModule implementations (calendar, maps, patient,
                     preferences, webview, helloworld, test modules)
```
