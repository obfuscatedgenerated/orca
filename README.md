<div align="center">
  <br />
  <p>
    <a href="https://cee-studio.github.io/orca"><img src="https://raw.githubusercontent.com/cee-studio/orca-docs/master/docs/source/images/logo.svg" width="546" alt="orca" style="background-color:red;" /></a>
  </p>
  <br />
  <p>
    Easy to reason, easy to debug, easy to use.
  </p>
  <p>
    Join our Discord server: <br> <a href="https://discord.gg/nBUqrWf"><img src="https://img.shields.io/discord/562694099887587338?color=5865F2&logo=discord&logoColor=white" alt="Discord server" /></a> </br>
  </p>
</div>

## About

Orca is implemented in plain C, its symbols are organized to be easily matched to the documentation of the API being covered.

This is done in order to:
* Minimize the need to thoroughly document every Orca API.
* Reduce our user's cognitive burden of having to read both Orca API documentation and supported REST API documentation.
* The codebase becomes easier to navigate.

Orca's implementation has minimum external dependencies to make bot deployment deadly simple.

### Design

- Easy to reason about the code: the most native data structures,
   the simplest algorithms, and intuitive interfaces.

- Easy to debug (networking and logic) errors: extensive assertion 
  and logging facilities.

- Easy to use for the end users: highly scalable, all transfers made with
  Orca are thread-safe.

### Minimal example

```c
#include <string.h> // strcmp()
#include <orca/discord.h>

void on_ready(struct discord *client) 
{
  const struct discord_user *bot = discord_get_self(client);
  log_info("Logged in as %s!", bot->username);
}

void on_message(struct discord *client, const struct discord_message *msg)
{
  if (strcmp(msg->content, "ping") != 0)
    return; // ignore messages that aren't 'ping'

  discord_async_next(client, NULL); // make next request non-blocking (OPTIONAL)
  struct discord_create_message_params params = { .content = "pong" };
  discord_create_message(client, msg->channel_id, &params, NULL);
}

int main(void)
{
  struct discord *client = discord_init(BOT_TOKEN);
  discord_set_on_ready(client, &on_ready);
  discord_set_on_message_create(client, &on_message);
  discord_run(client);
}
```
*This is a minimalistic example, refer to [`examples/`](examples/) for a better overview.*

## Build Instructions

### On Windows

#### With Windows Subsystem for Linux

* Install **WSL2** and get either Ubuntu or Debian [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
* **Make sure you are in your Linux $HOME folder before proceeding!**
* Continue to [On Linux](#on-linux) and follow your distro's building steps.

#### With MinGW

* Install MinGW64 from [MSYS2](https://www.msys2.org/).
* Open a MinGW64 terminal (blue background) and install the dependencies:
```bash
$ pacman -S pacman -S mingw-w64-x86_64-curl-winssl
```

* If you don't have them, you'll also need GCC and Make (mingw32-make will not work):
```bash
$ pacman -S mingw-w64-x86_64-gcc make
```

* You can CD into your filesystem to prepare to clone orca. For example, to go to your Desktop:
```bash
$ cd /c/Users/YourUserName/Desktop
```

* Clone the repository:
```bash
$ git clone https://github.com/cee-studio/orca.git && cd orca
```

* Still within this terminal (or another if you have added the appropriate paths to your PATH environment variable), build orca within its directory:
```bash
$ make
```

* If you want to work outside of the `my_bot` directory, install the library:
```bash
$ make install PREFIX=/mingw64
```

* You can continue reading these docs for usage examples, ignoring the Linux-specific parts (i.e. start from [*Configuring orca*](#configuring-orca) and ignore the `sudo make install` command in the *Installing orca* section).

* **NOTE: you need to link to the ws2_32 library when compiling your bot. You can pass the -lws2_32 argument to GCC or configure this within your Makefile/CMakeLists.txt alongside linking to discord and curl as stated later. For example, in the Makefile in the `my_bot` directory, add -lws2_32 to the end of LDFLAGS (line 52).**

### On Linux

The only dependency is `curl-7.4.1` or higher

#### Ubuntu and Debian

```bash
$ sudo apt install -y build-essential libcurl4-openssl-dev
```

#### Void Linux

```bash
$ sudo xbps-install -S libcurl-devel
```
### Setting up your environment

#### Clone orca into your workspace

```bash
$ git clone https://github.com/cee-studio/orca.git && cd orca
```

#### Compile orca

```bash
$ make
```

### Configuring orca

The following outlines the default fields of `config.json`
```js
{
  "logging": { // logging directives
    "level": "trace",        // trace, debug, info, warn, error, fatal
    "filename": "bot.log",   // the output file
    "quiet": false,          // change to true to disable logs in console
    "overwrite": false,      // overwrite existing file with "filename"
    "use_color": true,       // log with color
    "http": {
      "enable": true,        // generate http specific logging
      "filename": "http.log" // the output file
    },
    "disable_modules": ["WEBSOCKETS", "USER_AGENT"] // disable logging for these modules
  },
  ...         // API directives (discord, slack, github, etc)
}
```

### Test Echo-Bot

1. Get your bot token and add it to `config.json`, 
   by assigning it to discord's "token" field. There are 
   well written instructions from the 
   [discord-irc](https://github.com/reactiflux/discord-irc/wiki/Creating-a-discord-bot-&-getting-a-token)
   about how to get your bot token and adding it to a server.
2. Build example executables:
   ```bash
   $ make examples
   ```
3. Run Echo-Bot:
   ```bash
   $ cd examples && ./bot-echo
   ```

#### Get Echo-Bot Response

Type a message in any channel the bot is part of and the bot should send an echo response in return.

#### Terminate Echo-Bot

With <kbd>Ctrl</kbd>+<kbd>c</kbd> or by closing the Terminal.

### Create your first bot

* Head to `my_bot/`, a special folder set-up for your convenience that may be modified freely.
* Read our guide for [building your first bot](docs/BUILDING_A_BOT.md).

## Installing orca

Orca can be installed in case developing inside of `my_bot/` doesn't suit your needs:
```bash
$ sudo make install
```

Included headers must be `orca/` prefixed:
```c
#include <orca/discord.h>
```

### Standalone executable

#### GCC (Recommended)

```bash
$ gcc myBot.c -o myBot -pthread -ldiscord -lcurl
```

#### Clang

```bash
$ clang myBot.c -o myBot -pthread -ldiscord -lcurl
```

## Recommended debuggers

First, make sure your executable is compiled with the `-g` flag to ensure human-readable debugger messages.

### Valgrind

Using valgrind to check for memory leaks:

```bash
$ valgrind --leak-check=full ./myBot
```
For a more comprehensive guide check [Valgrind's Quick Start](https://valgrind.org/docs/manual/quick-start.html).

### GDB

Using GDB to check for runtime errors, such as segmentation faults:

```bash
$ gdb ./myBot
```
And then execute your bot from the gdb environment:
```bash
(gdb) run
```
If the program has crashed, get a backtrace of the function calls leading to it:
```bash
(gdb) bt
```

For a more comprehensive guide check [Beej's Quick Guide to GDB](https://beej.us/guide/bggdb/)

## Support

Problems? Check out our [Discord Server](https://discord.gg/nBUqrWf).

## Links

- [Documentation](https://cee-studio.github.io/orca/)
- [Building your first bot](docs/BUILDING_A_BOT.md)
- [Contributing](docs/CONTRIBUTING.md)
<!-- - [Internals](docs/INTERNALS.md) -->

## Contributing
Check our [Contributing Guidelines](docs/CONTRIBUTING.md) to get started! If you are here for the Discord API, please check our [Discord API Roadmap](docs/DISCORD_ROADMAP.md).

**Give us a star if you like this project!**
