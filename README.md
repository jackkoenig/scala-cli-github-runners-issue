Scala CLI does not seem to work with the default Java 17 (nor Java 21) on Github ubuntu-22.04 runners.

See [.github/workflows/test.yml](.github/workflows/test.yml), pasted here:
```yml
name: Possible Scala CLI Issue

on:
  workflow_dispatch:

jobs:
  works:
    name: Show that Scala CLI works with JVM it downloads
    runs-on: ubuntu-22.04
    steps:
      - uses: VirtusLab/scala-cli-setup@v1
      - run: echo 'println("Hello")' | scala-cli _.sc

  fails:
    name: Show that Scala CLI fails with System JVM
    runs-on: ubuntu-22.04
    steps:
      - name: Setup Java
        run: echo "JAVA_HOME=${JAVA_HOME_17_X64}" >> "$GITHUB_ENV"
      - uses: VirtusLab/scala-cli-setup@v1
        with:
          jvm: '' # Deliberately empty to avoid downloading JVM
      - run: echo 'println("Hello")' | scala-cli _.sc

  workaround:
    name: Show workaround with --server=false
    runs-on: ubuntu-22.04
    steps:
      - name: Setup Java
        run: echo "JAVA_HOME=${JAVA_HOME_17_X64}" >> "$GITHUB_ENV"
      - uses: VirtusLab/scala-cli-setup@v1
        with:
          jvm: '' # Deliberately empty to avoid downloading JVM
      - run: echo 'println("Hello")' | scala-cli --server=false _.sc
```

You can see a run of these 3 jobs [here](https://github.com/jackkoenig/scala-cli-github-runners-issue/actions/runs/10118228994).

The `fails` action fails with the following exception:
```
Starting compilation server
Exception in thread "main" java.util.concurrent.TimeoutException: Future timed out after [30 seconds]
	at scala.concurrent.impl.Promise$DefaultPromise.tryAwait0(Promise.scala:248)
	at scala.concurrent.impl.Promise$DefaultPromise.result(Promise.scala:261)
	at scala.concurrent.Await$.$anonfun$result$1(package.scala:201)
	at scala.concurrent.BlockContext$DefaultBlockContext$.blockOn(BlockContext.scala:62)
	at scala.concurrent.Await$.result(package.scala:124)
	at bloop.rifle.internal.Operations$.timeout(Operations.scala:532)
	at bloop.rifle.internal.Operations$.about(Operations.scala:500)
	at bloop.rifle.BloopRifle$.getCurrentBloopVersion(BloopRifle.scala:156)
	at bloop.rifle.BloopServer$.ensureBloopRunning(BloopServer.scala:111)
	at bloop.rifle.BloopServer$.bsp(BloopServer.scala:155)
	at bloop.rifle.BloopServer$.buildServer(BloopServer.scala:185)
	at scala.build.compiler.BloopCompilerMaker.$anonfun$1(BloopCompilerMaker.scala:48)
	at scala.build.compiler.BloopCompiler.<init>(BloopCompiler.scala:15)
	at scala.build.compiler.BloopCompilerMaker.$anonfun$2(BloopCompilerMaker.scala:51)
	at scala.util.Try$.apply(Try.scala:217)
	at scala.build.compiler.BloopCompilerMaker.create(BloopCompilerMaker.scala:51)
	at scala.build.compiler.ScalaCompilerMaker.withCompiler(ScalaCompilerMaker.scala:34)
	at scala.build.compiler.ScalaCompilerMaker.withCompiler$(ScalaCompilerMaker.scala:9)
	at scala.build.compiler.BloopCompilerMaker.withCompiler(BloopCompilerMaker.scala:14)
	at scala.build.Build$.build$$anonfun$3(Build.scala:616)
	at scala.build.EitherCps$Helper.apply(EitherCps.scala:19)
	at scala.build.Build$.build(Build.scala:619)
	at scala.cli.commands.run.Run$.runCommand(Run.scala:331)
	at scala.cli.commands.default.Default.runCommand(Default.scala:61)
	at scala.cli.commands.default.Default.runCommand(Default.scala:40)
	at scala.cli.commands.ScalaCommand.run(ScalaCommand.scala:379)
	at scala.cli.commands.ScalaCommand.run(ScalaCommand.scala:361)
	at caseapp.core.app.CaseApp.main(CaseApp.scala:157)
	at scala.cli.commands.ScalaCommand.main(ScalaCommand.scala:346)
	at caseapp.core.app.CommandsEntryPoint.main(CommandsEntryPoint.scala:169)
	at scala.cli.ScalaCliCommands.main(ScalaCliCommands.scala:125)
	at scala.cli.ScalaCli$.main0(ScalaCli.scala:292)
	at scala.cli.ScalaCli$.main(ScalaCli.scala:118)
	at scala.cli.ScalaCli.main(ScalaCli.scala)
```
