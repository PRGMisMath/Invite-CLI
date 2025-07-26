# Invite-CLI

This single-header librairy aims at providing an easy way of integrating an interactive terminal in your applications for testing purposes. 
It is design to make creating new command as easy as possible while maintaining maximum flexibility.

## Creating a terminal

To create a terminal, you just need to run the following line of code :

```cpp
Terminal term{};
```

To run it, just type :

```cpp
term.run();
```

By default, an `exit` and an `help` are implemented.

## Adding new commands

To have the maximum of flexibility, commands are classes inhering the `Command` class.
The parameters of the command are fields of the command class and there value can easily be used thanks to the `.value` attribute.

To create a new class, you must override the following methods : 
 - `name` : define the command name use in the terminal
 - `description` : define the description used in the `help` command
 - `execute` : what the command while execute
 - `configure` : add the parameters for serialization

### Passing parameters

To add a parameter, you need to add him as a field of the class command.
Then in the `configure` command, you need to pass it to the format manager and indicate if it is a required or optionnal parameters.
The order in which you pass your parameters matters : it is the one that will be used in the terminal.

```cpp
format.addReq(&n1)    // Required
      .addReq(&n2)    // Required
      .addOpt(&mult); // Optionnal
```

### Default value for parameters

You can also specify a default value for a parameter as follow : 

```cpp
n2.giveDefValue(0);
```

### Optionnal parameters

In the terminal, to use an optionnal parameters, you put a `-` before the name of the parameter.

For a flag (boolean) parameter, you just do `-mult`.
For other parameters, you do `-title "the title"`.

To see if a parameter has been set, use the implicit boolean conversion :

```cpp
if (param) {
  // The code when the param is set
}
```

This also works for flag. It is therefore useless to access the `.value` of a flag parameter.

## Example code

```cpp
#include<iostream>

#include "terminal.hpp"


class OpCMD : public Command {
public:
    OpCMD() :
        n1("n1","nombre 1"), n2("n2", "nombre 2"),
        mult("mult", "activer la multiplication"),
        title("title", "titre de l'addition")
    {
    }
    const char* name() const override { return "add"; }
    const char* description() const override { return "additionne 2 chiffres"; }
    void execute(TerminalState& state) override {
        if (title)
            *state.fluxOut << title.value << " : ";

        if (!mult)
            *state.fluxOut << n1.value + n2.value << std::endl;
        else
            *state.fluxOut << n1.value * n2.value << std::endl;
    }

protected:
    virtual void configure(CmdFormat& format) {
        n2.giveDefValue(0);
        format.addReq(&n1).addReq(&n2).addOpt(&mult).addOpt(&title);
    }

    IntParam n1, n2;
    BoolParam mult;
    StringParam title;
};

int main() {
    Terminal term{};
    term.addCommand(new OpCMD());
    term.run();
}
```
