# Change Log

## [25.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/25.0.0) (2026-07-26)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/21.0.0...25.0.0)

**Breaking changes:**

- Java 25 is now the minimum requirement, raised from Java 21. The artifact version follows the
  supported Java release, so this line continues as `25.x` instead of `21.x`.
- Update JavaFX from 21.0.12 to 26.0.2

**Implemented enhancements:**

- Drop the `org.testfx:openjfx-monocle` test dependency. Since version 26 JavaFX ships its own
  headless Glass platform, so the headless test setup no longer needs a third-party one. The
  Surefire configuration now selects it with `-Dglass.platform=Headless` instead of
  `-Dglass.platform=Monocle -Dmonocle.platform=Headless`. This also unpins JavaFX, which could
  not move past the 21 line while Monocle was needed — its last release is 21.0.2.
- Update CalendarFX from 11.12.7 to 12.1.0, which requires JavaFX 23 or newer and was therefore
  out of reach before
- Build and release on JDK 25 in the GitHub Actions workflows
- Document both supported lines in `README.md`, with a table mapping the WorkbenchFX version to
  the Java and JavaFX versions it needs

**Fixed bugs:**

- Manage `commons-logging`, which CalendarFX 12.1.0 pulls in at three different versions through
  its own dependency and through the `commons-validator` / `commons-beanutils` chain. This broke
  the `DependencyConvergence` enforcer rule in the demo.

**Known issues:**

- The WebView WebSocket `UnsatisfiedLinkError` reported for 21.0.0 is still present. JavaFX 26
  declares `twkDidOpen`, `twkDidReceiveData`, `twkDidFail` and `twkDidClose` as native methods on
  `com.sun.webkit.network.SocketStreamHandle`, but the bundled `libjfxwebkit` implements none of
  them. Moving to a newer JavaFX therefore does not fix it. See the 21.0.0 entry below and
  `workbenchfx-demo/README.md` for details.
- `testfx.headless=true` must not be set. TestFX 4.0.18 implements that flag by loading the
  Monocle platform factory, so setting it fails with a `ClassNotFoundException` on the headless
  JavaFX platform. Headlessness comes from `glass.platform` instead.
- The Surefire flags have to stay in `systemPropertyVariables` and cannot move into `argLine`,
  because `forkCount` is `0` and Surefire ignores `argLine` when it does not fork.

## [21.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/21.0.0) (2026-07-26)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/11.3.1...21.0.0)

**Breaking changes:**

- Java 21 is now the minimum requirement, raised from Java 11. The artifact version follows the
  supported Java release, so this line continues as `21.x` instead of `11.x`.
- Update JavaFX from 17.0.1 to 21.0.12
- Update SLF4J from 1.7.32 to 2.0.18. The 1.7 and 2.x bindings are not interchangeable, so
  applications that supply their own binding have to switch to an SLF4J 2.x one — for Log4j that
  means replacing `log4j-slf4j-impl` with `log4j-slf4j2-impl`, as the demo now does.

**Implemented enhancements:**

- Update Monocle to 21.0.2 to match the JavaFX version used by the headless tests
- Update the test stack so it can read Java 21 class files: Spock 2.4 on Groovy 5.0,
  JUnit 6.1.2 (Jupiter, Vintage and Platform), TestFX 4.0.18, Mockito 5.23.0,
  Byte Buddy 1.18.11, Hamcrest 3.0, Objenesis 3.5, AssertJ 3.27.7, Awaitility 4.3.0,
  GMavenPlus 5.1.0 and Surefire 3.5.6
- Update the remaining libraries to their latest versions compatible with the JavaFX 21 baseline:
  Guava 33.6.0-jre, Log4j 2.26.1, Ikonli 12.4.0, ControlsFX 11.2.3, PreferencesFX 11.19.0,
  CalendarFX 11.12.7, GMapsFX 11.0.7 and CSSFX 11.5.1
- Update the build plugins: Compiler 3.15.0, Source 3.4.0, Javadoc 3.12.0, Surefire 3.5.6,
  JaCoCo 0.8.15, Exec 3.6.3 and Build Helper 3.6.1. The Javadoc plugin no longer overrides ASM
  with 8.0.1, a version that predates and cannot read Java 21 class files, but uses 9.10.1.
- Declare every dependency and plugin version through a property. The properties live in the
  parent POM, except for the ones only the demo uses, which live in `workbenchfx-demo/pom.xml`.
- Update `dlsc-maven-parent` to 1.6.0 and the Maven wrapper to 3.9.11
- Run the JavaFX tests headless on Monocle
- Build and release on JDK 21 in the GitHub Actions workflows
- Document the demo applications and how to run them in `workbenchfx-demo/README.md`

**Fixed bugs:**

- `WorkbenchSpec` stubbed the final methods `WorkbenchModule.getName()` and
  `WorkbenchModule.getIcon()`, which silently had no effect, so every mock module was named `""`
  during the tests. The name is now passed to the real constructor, and the two dead stubs are
  gone. Spock 2.4 rejects such stubs instead of ignoring them, which is how this surfaced.
- `SelectionStripSpec` checked `instanceof Callback<SelectionStrip, StripCell<WorkbenchModule>>`.
  The type arguments were erased at runtime and never took part in the check; Groovy 5 rejects
  such an expression outright instead of accepting it, so the assertion now names the raw type.
- Manage `ikonli-material-pack` alongside the other Ikonli artifacts. PreferencesFX pulls it in
  transitively at an older version than the one the rest of the Ikonli dependencies resolve to,
  which broke the `requireUpperBoundDeps` enforcer rule in the demo.

**Known issues:**

- WebView pages that open a WebSocket fail with
  `UnsatisfiedLinkError: 'void com.sun.webkit.network.SocketStreamHandle.twkDidOpen(long)'`.
  This is an upstream JavaFX problem: since 21.0.3 the bundled `libjfxwebkit` no longer
  implements the `SocketStreamHandle` native methods, although the Java class still declares and
  calls them. The last release with a working implementation is JavaFX 21.0.2, and the newest
  JavaFX line is affected as well, so upgrading is not a way out. In the demos this shows up when
  opening the *JFX-Central* module. See `workbenchfx-demo/README.md` for details.
- JavaFX stays on the 21 line even though newer releases exist, because Monocle — which the
  headless tests run on — has no release beyond 21.0.2. CalendarFX is held at 11.12.7 for the
  same reason: 12.x requires JavaFX 23 or newer.

## [11.0.2](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/11.0.2) (2019-09-08)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/8.0.2...11.0.2)

## [8.0.2](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/8.0.2) (2019-09-08)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/11.0.1...8.0.2)

**Merged pull requests:**

- Bump preferencesfx-core from 11.5.0 to 11.6.0 [\#67](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/67) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump preferencesfx-core from 8.5.0 to 8.6.0 [\#66](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/66) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump awaitility from 4.0.0 to 4.0.1 [\#65](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/65) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump awaitility from 4.0.0 to 4.0.1 [\#64](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/64) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump checkstyle from 8.23 to 8.24 [\#62](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/62) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump checkstyle from 8.23 to 8.24 [\#61](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/61) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump asm from 7.0 to 7.1 [\#60](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/60) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump javafx-swing from 11.0.1 to 12.0.2 [\#59](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/59) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump javafx-fxml from 11.0.1 to 12.0.2 [\#57](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/57) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump javafx-controls from 11.0.1 to 12.0.2 [\#56](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/56) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump view from 11.5.0 to 11.6.4 [\#54](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/54) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump javafx-web from 11.0.1 to 12.0.2 [\#52](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/52) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump preferencesfx-core from 11.3.0 to 11.5.0 [\#51](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/51) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump view from 8.6.0 to 8.6.1 [\#50](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/50) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump controlsfx from 8.40.14 to 8.40.15 [\#49](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/49) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump view from 8.5.0 to 8.6.0 [\#48](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/48) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump checkstyle from 8.22 to 8.23 [\#46](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/46) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump preferencesfx-core from 8.3.0 to 8.5.0 [\#45](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/45) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump log4j-api from 2.12.0 to 2.12.1 [\#43](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/43) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump slf4j-api from 1.7.26 to 1.7.28 [\#42](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/42) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump testfx-junit5 from 4.0.15-alpha to 4.0.16-alpha [\#39](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/39) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump assertj-core from 3.13.1 to 3.13.2 [\#37](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/37) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump byte-buddy from 1.9.16 to 1.10.1 [\#36](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/36) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump awaitility from 3.1.6 to 4.0.0 [\#35](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/35) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump log4j-slf4j-impl from 2.12.0 to 2.12.1 [\#33](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/33) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump log4j-core from 2.12.0 to 2.12.1 [\#32](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/32) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump guava from 28.0-jre to 28.1-jre [\#29](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/29) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump testfx-spock from 4.0.15-alpha to 4.0.16-alpha [\#28](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/28) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Bump testfx-core from 4.0.15-alpha to 4.0.16-alpha [\#25](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/25) ([dependabot-preview[bot]](https://github.com/apps/dependabot-preview))
- Add license scan report and status [\#24](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/24) ([fossabot](https://github.com/fossabot))

## [11.0.1](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/11.0.1) (2019-07-30)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/8.0.0...11.0.1)

## [8.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/8.0.0) (2019-07-30)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/v11.0.0...8.0.0)

**Implemented enhancements:**

- Update dependency versions [\#22](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/22) ([martinfrancois](https://github.com/martinfrancois))
- Deploy CI/CD with Travis CI [\#21](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/21) ([martinfrancois](https://github.com/martinfrancois))
- \#9 for Java 11 [\#19](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/19) ([martinfrancois](https://github.com/martinfrancois))
- \#9 for Java 10 [\#18](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/18) ([martinfrancois](https://github.com/martinfrancois))
- Update dependencies [\#17](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/17) ([martinfrancois](https://github.com/martinfrancois))

**Fixed bugs:**

- Fix readme images [\#11](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/11) ([martinfrancois](https://github.com/martinfrancois))

**Closed issues:**

- log4j-to-slf4j? [\#8](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/issues/8)
- Strategy to init WorkbenchModule [\#6](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/issues/6)
- ControlsFX [\#5](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/issues/5)

**Merged pull requests:**

- Fix Tests Java 11 [\#16](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/16) ([martinfrancois](https://github.com/martinfrancois))
- Fix Java 10 Tests [\#15](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/15) ([martinfrancois](https://github.com/martinfrancois))
- Fix Java 10 build [\#13](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/13) ([martinfrancois](https://github.com/martinfrancois))
- Make it possible to execute demo with Java 11 [\#12](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/12) ([martinfrancois](https://github.com/martinfrancois))
- Remove documentation [\#10](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/10) ([martinfrancois](https://github.com/martinfrancois))
- update version to 1.0.1-SNAPSHOT, remove log4j and logback [\#9](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/9) ([sirolf2009](https://github.com/sirolf2009))

## [v11.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/v11.0.0) (2018-09-24)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/v10.0.0...v11.0.0)

## [v10.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/v10.0.0) (2018-09-24)
[Full Changelog](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/compare/v1.0.0...v10.0.0)

**Fixed bugs:**

- Added java-11 support to your project and replaced log4j with slf4j-based logger. [\#2](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/2) ([markehammons](https://github.com/markehammons))

**Closed issues:**

- Please tag commit 67a32cf as v1.0.0 [\#1](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/issues/1)

**Merged pull requests:**

- Corrections for new Repository [\#4](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/pull/4) ([martinfrancois](https://github.com/martinfrancois))

## [v1.0.0](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX/tree/v1.0.0) (2018-09-12)


\* *This Change Log was automatically generated by [github_changelog_generator](https://github.com/skywinder/Github-Changelog-Generator)*