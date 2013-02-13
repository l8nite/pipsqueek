How to extend !PipSqueek by writing a module.

= Introduction =

PipSqueek is built on [http://poe.perl.org/ the POE framework]. It utilises a plugin system similar to that of Perl itself. Extra functionality can be added to !PipSqueek by adding new modules. This page aims to teach you how to write such a module.


= Writing the module =

To begin with I'll assume you have !PipSqueek installed and running in a folder you have write access to. If you navigate to the _lib/!PipSqueek/Plugin_ directory under your main !PipSqueek directory you should see some files listed in there with a .pm extension.

In this folder you should create another file named _Random.pm_ and open it in your [http://vim.org favourite text editor].

The first things we need to do are to name the module, and tell it to include the base !PipSqueek::Plugin module.

{{{
package PipSqueek::Plugin::Random;
use base qw(PipSqueek::Plugin);
}}}

The first line above gives this module a name that is not currently in use in the !PipSqueek namespace: !PipSqueek::Plugin::Random. _!PipSqueek::_ is the namespace, _Plugin::_ is a more specific area of the !PipSqueek namespace stating that this is a plugin, and _Random_ is the actual name of the module. _*Do not call your module "Random" if you already have a module by that name. By default you won't have this module.*_

Next you need to initialise the plugin.

{{{
sub plugin_initialize {
  my $self = shift;

  $self->plugin_handlers({
      'multi_random'    => 'random_number',
  });
}
}}}

To set up plugin handlers there is a function named, somewhat predictably, _plugin_handlers_. This is where you explain to !PipSqueek that your module should respond when someone says _!random_. The _multi_random_ specifies that !PipSqueek should respond in both a channel and in a private message. The _random_number_ part is the name of the sub that should be called when someone says _!random_.

If you required that your module was only accessible in a private message then you should use _private_random_ instead. If your module should only be used in public then you should use _public_random_.

Now you have told !PipSqueek where to look when your module is called you need to tell it what to do.

{{{
sub random_number {
  my ($self, $message) = @_;
  return $self->respond($message, int(rand(100)));
}

1;
}}}

This tells !PipSqueek to exit the function by responding to the request. _$message_ is the first argument, and what you want !PipSqueek to respond with is the second. In this case we're telling !PipSqueek to respond with a random number between 0 and 99.

The very final line is _1;_ because Perl complains if the very last statement executed in a module is not parsed as true. Make sure you do not put any code after this point.

You can now save the file. You have a basic module that returns a random number between 0 and 99 when someone says _!random_.

== Accepting input ==

This is pretty limited for a module. It'd be better if the user could specify a range for the number to be between. Let's take a look at accepting command input. The section below will replace the _sub random_number_ section we wrote above.

{{{
sub random_number {
  my ($self, $message) = @_;
  my $range = $message->command_input();

  my ($from, $to) = $range =~ m/^\s*([\d]+)[\s-]{1}([\d]+)\s*$/;
  my $spread = $to-$from;

  my $output = int(rand($spread)) + $from;

  return $self->respond($message, $output);
}

1;
}}}

The code above uses the _$message->command_input()_ method to retrieve whatever the user typed after the _!random_ command. In this case it's looking for two unsigned integers separated either by a space or a hyphen. These numbers specify the range that the random number should be within.

There are limitations to this module as it is only an example, but for the sake of keeping things clear I won't go into them here.


= The complete module =

Here is a copy of everything you should have. You can just copy and paste this if you prefer to learn that way.

{{{
package PipSqueek::Plugin::Random;
use base qw(PipSqueek::Plugin);

sub plugin_initialize {
  my $self = shift;

  $self->plugin_handlers({
      'multi_random'    => 'random_number',
  });
}

sub random_number {
  my ($self, $message) = @_;
  my $range = $message->command_input();

  my ($from, $to) = $range =~ m/^\s*([\d]+)[\s-]{1}([\d]+)\s*$/;
  my $spread = $to-$from;

  my $output = int(rand($spread)) + $from;

  return $self->respond($message, $output);
}

1;
}}}


= Testing the module =

To test the module you will need to have !PipSqueek running and connected to an IRC server. Connect to the the server, join a channel with your !PipSqueek bot. If !PipSqueek was running before you copied in the module then you will need to _!rehash_ first. You can now type _!random 5 10_. !PipSqueek should respond with a random number between 5 and 10.


= Documenting the module =

All good modules have documentation and help files for its users. Yours should be no different. Write a file similar to this one. Saving it as _doc/plugins/Random/multi_random_ will allow users of !PipSqueek to type _!help random_ to read the help you have provided.

{{{
random <from> <to>

The bot will return a random number between <from> and <to>.
}}}


= Share =


If you've written a really cool module then don't keep it to yourself. Send a message letting us know about it and it may even be included in the next release of !PipSqueek.
