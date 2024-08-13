# Recommended Tools and Apps

If you're new to macOS or JavaScript development, figuring out which apps to use can be overwhelming. Here's a list of our favorites. Since app preferences vary, feel free to pick and choose what works best for you. And if you have any cool recommendations of your own, we'd love to hear them!

---

## Package management

A good package manager can make your life a whole lot easier, but since macOS doesn't come with a built-in package manager like some other Unix/Linux distributions we will need to install one.

### Homebrew

One of the most popular package managers for Mac is [Homebrew](https://brew.sh/).
To install it just run the following command in your terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Now that you have Homebrew installed, you can check if everything went well by running:

```bash
brew doctor
```

#### So how do I use it?

There are two ways to install packages use Homebrew. The command to use will depend on the type of package you want to install.
        
For CLI (Command Line Interface) packages you can simply run:

=== "Command"

    ```bash
    brew install <package-name>
    ```

=== "Example"

    ```bash
    # For example if you want to install wget

    brew install wget
    ```

For GUI (Graphical User Interface) packages we need to specify in the command that we want to install a Cask, not a Formulae. For that you can run:

=== "Command"

    ```bash
    brew install --cask <package-name>
    ```

=== "Example"

    ```bash
    # For example if you want to install iterm2

    brew install --cask iterm2
    ```

### Alternatives

- [Macports](https://www.macports.org/)
- [Nix](https://nixos.org/)

---

## Terminal

The default macOS terminal app has its limitations, so you might want to explore more featured and customizable alternatives.

### iTerm2   

One of the most popular choices is [iTerm2](https://iterm2.com/). Installation is straightforward: simply download the binary from their [downloads page](https://iterm2.com/downloads.html) and move it to your `/Applications` folder.

Alternatively, if you prefer using Homebrew (which we previously installed) to manage desktop apps, you can install iTerm2 with the following command:

```bash
brew install --cask iterm2
```

### Alternatives

- [Alacritty](https://alacritty.org/)
- [Kitty](https://sw.kovidgoyal.net/kitty/)
- [Hyper](https://hyper.is/)

---

## Shell

A Unix shell is a command-line interpreter that provides a command line interface for UNIX systems like your MacBook. There are different shells available, so you can choose the one you prefer.

One of the most well-known is BASH (Bourne Again Shell), which is the default shell on most Linux distributions and was the default on macOS until the release of macOS Catalina (10.15). Since then, the default shell on macOS is zsh. We'll assume this is the one you have installed and set as default.

To check your default shell, run:
```bash
echo $0
```
    
You can choose to use the shell of your choice and follow the install steps for the one you pick.
        
### Zsh (Z Shell)

If you don't have zsh (and want to switch to it) here's how to do it:

```bash
brew install zsh && chsh -s $(which zsh)
```
       
### Oh My Zsh
A great way to enhance your zsh configuration is by installing [Oh My Zsh](https://ohmyz.sh/). Oh My Zsh is a framework to manage your zsh configuration. You can read more about it [here](https://github.com/ohmyzsh/ohmyzsh/wiki)

To install Oh My Zsh, run:
```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

The installation will create the necessary configuration files, which you can edit as needed.

To open your configuration file run:
```bash
open ~./zsh
```

### Alternatives
- [Fish](https://fishshell.com/)
- [Korn](http://kornshell.com/)   

---

## IDE
An Integrated Development Environment (IDE) is essential for coding and managing your projects efficiently. There are several IDEs available, but if you are not sure what to use we recommend using Visual Studio Code (VS Code).

To install VS Code you can download it from the [official website](https://code.visualstudio.com/Download) or install it via Homebrew:

```bash
brew install --cask visual-studio-code
```

### Alternatives
- [VSCodium](https://vscodium.com/)
- [Sublime Text](https://www.sublimetext.com/)

---

## MongoDB GUI

For managing MongoDB databases, we recommend MongoDB Compass, the official GUI. It offers an easy-to-use interface to visualize, query, and manage your databases.

To install it, download the binary from their [downloads page](https://www.mongodb.com/try/download/compass) and move it to your `/Applications` folder.

### Alternatives
- [Studio 3T](https://studio3t.com/)

---

## API Client

For testing and interacting with APIs, we recommend Postman, since it is the tool used by most people, making it easier to share configurations across team members.

To install it, download the binary from their [downloads page](https://www.postman.com/downloads/) and move it to your `/Applications` folder.

### Alternatives
- [Insomnia](https://insomnia.rest/products/insomnia)
