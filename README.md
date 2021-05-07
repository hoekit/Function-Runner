# NAME

Function::Runner - Define functions at a higher level and run them

# SYNOPSIS

    use Function::Runner;

    # Hello World
    sub greet {
        print "Hello ". ($_[0] || 'World') ."\n";
        return ('ok',$_[0]);
    }

    my $defn = {                              # Definition is just a hashref
      '/hello' => '&greet'                    #   The /hello step,
    };                                        #     calls the &greet function

    my $fn = Function::Runner->new($defn);    # Create a greeter
    $fn->run('/hello','Flash');               # Hello Flash


    my $switch = {                            # Define a switch
      '/checkSwitch' => {
          'run'  => '&checkSwitch',           # Check the switch
          ':on'  => '&bye',                   #   If it's on, leave
          ':off' => '/turnOn',                #   If it's off, turn it on
      },
      '/turnOn'  => {                         # Turn on the switch
          'run'  => '&greet',                 #   Greet the caller
          ':ok' => '/turnOff',                #   Then turn off the switch
      },
      '/turnOff' => '&bye',                   # Turn off the switch and leave
    };
    sub bye {
      print "Bye ". ($_[0] || 'World') ."\n";
      return ('ok',$_[0]);
    }
    sub checkSwitch { return shift }

    $fn = Function::Runner->new($switch);     # Create a switch
    $fn->run('/checkSwitch', 'on', 'Flash');  # Bye Flash

    $fn->run('/checkSwitch', 'off', 'Hulk');  # Hello Hulk
                                              # Bye Hulk

    say join ' ', @$_ for @{$fn->steps};      # List steps, function and result
                                              #   /checkSwitch &checkSwitch :off
                                              #   /turnOn &greet :ok

# DESCRIPTION

Function::Runner provides a way to define the steps of a function steps
logical flow between the steps using just hashrefs. The user then
implements the steps that need to be called. The function runner will
then run the function.

This module is handy for functions that are naturally composed of many
hierarchical steps and flows differently depending on the results of
those steps. The function definition helps to clarify that hierarchy and
flow at a higher level.

A function definition (**funcdef**) is composed of three (3) constructs:
_steps_, _functions_ and _results_. Each construct is a string with a
different character prefix to indicate the kind of construct:

    /a_step         # Steps are prefixed with /, like directories

    &a_function     # Functions prefixed with &, like Perl

    :some_result    # Results prefixed with :

The keys of the funcdef hashref is always a _step_. The value of the
funcdef hashref is the step definition (**stepdef**) defines how that step
is to be executed.

A _stepdef_ can be just a _function_ if no further steps follow. For
example:

    { '/hello' => '&greet' }

A _stepdef_ can also be a hashref that defines the _function_ to run
and the _next step_ to take depending on the _results_ of that
function run.  For example:

    '/checkSwitch' => {
        'run'  => '&checkSwitch',           # Check the switch
        ':on'  => '&bye',                   #   If it's on, leave
        ':off' => '/turnOn',                #   If it's off, turn it on
    },

The next step can either be a function:

    ':on'  => '&bye'            # On "on" result, call the &bye function

or it can be another step:

    ':off' => '/turnOn'         # On "off" result, run the /turnOn step

# METHODS

## run($step,@args)

The run() method runs the given $step, checks the results, looks up the
function definition to determine the next step to run an calls that
until there is nothing left to be done at which point it will return the
result of the last function that was called.

Along the way it tracks how deep it is within the function definition.
Each step that was ran and the corresponding result is stored in an
array.

## steps()

The steps() method returns the steps that ran for that function.

# NAMING CONVENTIONS

_Steps_ and _Functions_ with the form **Verb + Object** is rather
pleasing to read. For example:

    &findStalledWorkers

    /find_stalled_workers

_Results_ with the form **Object + State** or **Object + Adjective** is
also rather pleasing to read. For example:

    :queueEmpty

    :not_found

Taken together, the step and it's results end up reading like this:

    /find_stalled_workers :not_found

# NOTES

Defining the a function in terms of _steps_, _functions_ and
_results_ has several nice properties.

The valid return values from each _function_ is clearly spelled out.

Having a function definition makes it easier rearrange the flow of a
function or add an additional step in the function's processing.

It is possible to directly call the steps in a function definition.

It is possible to analyze the _funcdef_ hashref to create a reverse
dependency graph so that when a _function_ is about to be changed, find
all it's dependents.

All in all, it is a rather nice way to design these kinds of
hierarchical, multi-step functions where the flow depends on the results
of prior steps.

# AUTHOR

Hoe Kit CHEW <hoekit@gmail.com>

# COPYRIGHT

Copyright 2021- Hoe Kit CHEW

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
