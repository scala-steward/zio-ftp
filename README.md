[//]: # (This file was autogenerated using `zio-sbt-website` plugin via `sbt generateReadme` command.)
[//]: # (So please do not edit it manually. Instead, change "docs/index.md" file or sbt setting keys)
[//]: # (e.g. "readmeDocumentation" and "readmeSupport".)


# ZIO FTP

[ZIO FTP](https://zio.dev) is a thin wrapper over (s)Ftp client for [ZIO](https://zio.dev).

Project Stage | CI | Release | Snapshot | Discord | Github |
--------------|----|---------|----------|---------|--------|
[![Production Ready](https://img.shields.io/badge/Project%20Stage-Production%20Ready-brightgreen.svg)](https://github.com/zio/zio/wiki/Project-Stages)        |![CI Badge](https://github.com/zio/zio-ftp/workflows/CI/badge.svg) |[![Sonatype Releases](https://img.shields.io/nexus/r/https/oss.sonatype.org/dev.zio/zio-ftp_2.12.svg)](https://oss.sonatype.org/content/repositories/releases/dev/zio/zio-ftp_2.12/) |[![Sonatype Snapshots](https://img.shields.io/nexus/s/https/oss.sonatype.org/dev.zio/zio-ftp_2.12.svg)](https://oss.sonatype.org/content/repositories/snapshots/dev/zio/zio-ftp_2.12/) |[![Chat on Discord!](https://img.shields.io/discord/629491597070827530?logo=discord)](https://discord.gg/2ccFBr4) |[![ZIO FTP](https://img.shields.io/github/stars/zio/zio-ftp?style=social)](https://github.com/zio/zio-ftp) |


## Installation

In order to use this library, we need to add the following line in our `build.sbt` file:

```scala
libraryDependencies += "dev.zio" %% "zio-ftp" % "0.1.0" 
```

## How to use it?

* Imports
```scala
import zio.ftp._
```

* FTP
```scala
// FTP
val unsecureSettings = UnsecureFtpSettings("127.0.0.1", 21, FtpCredentials("foo", "bar"))

//listing files
Ftp.ls("/").runCollect.provideLayer(unsecure(unsecureSettings))
```

* FTPS
```scala
// FTPS
val secureSettings = SecureFtpSettings("127.0.0.1", 21, FtpCredentials("foo", "bar"))

//listing files
SFtp.ls("/").runCollect.provideLayer(secure(secureSettings))
```

* SFTP (support ssh key)

```scala
val sftpSettings = SecureFtpSettings("127.0.0.1", 22, FtpCredentials("foo", "bar"))

//listing files
SFtp.ls("/").runCollect.provideLayer(secure(sftpSettings))
```

## Example

First we need an FTP server, so let's create one:

```bash
docker run -d \
    -p 21:21 \
    -p 21000-21010:21000-21010 \
    -e USERS="one|1234" \
    -e ADDRESS=localhost \
    delfer/alpine-ftp-server
```

Now we can run the example:

```scala
import zio._
import zio.stream._
import zio.ftp._
import zio.ftp.Ftp._

import java.io.IOException

object ZIOFTPExample extends ZIOAppDefault {

  private val settings =
    UnsecureFtpSettings("127.0.0.1", 21, FtpCredentials("one", "1234"))

  private val myApp: ZIO[Scope with Ftp, IOException, Unit] =
    for {
      _        <- Console.printLine("List of files at root directory:")
      resource <- ls("/").runCollect
      _        <- ZIO.foreach(resource)(e => Console.printLine(e.path))
      path      = "~/file.txt"
      _        <- upload(
                    path,
                    ZStream.fromChunk(
                      Chunk.fromArray("Hello, ZIO FTP!\nHello, World!".getBytes)
                    )
                  )
      file     <- readFile(path)
                    .via(ZPipeline.utf8Decode)
                    .runCollect
      _        <- Console.printLine(s"Content of $path file:")
      _        <- Console.printLine(file.fold("")(_ + _))
    } yield ()

  override def run = myApp.provideSomeLayer(unsecure(settings))
}
```

## Support any commands?

If you need a method which is not wrapped by the library, you can have access to underlying FTP client in a safe manner by using

```scala
import zio._

trait FtpAccessors[+A] {
  def execute[T](f: A => T): ZIO[Any, IOException, T]
} 
```

## Documentation

Learn more on the [ZIO FTP homepage](https://zio.dev/zio-ftp/)!

## Contributing

For the general guidelines, see ZIO [contributor's guide](https://zio.dev/about/contributing).

## Code of Conduct

See the [Code of Conduct](https://zio.dev/about/code-of-conduct)

## Support

Come chat with us on [![Badge-Discord]][Link-Discord].

[Badge-Discord]: https://img.shields.io/discord/629491597070827530?logo=discord "chat on discord"
[Link-Discord]: https://discord.gg/2ccFBr4 "Discord"

## License

[License](LICENSE)
