[![Build status](https://ci.appveyor.com/api/projects/status/aux2774o26tohc81?svg=true)](https://ci.appveyor.com/project/Siliconrob/commandline-options) 
[![CircleCI](https://circleci.com/gh/Siliconrob/CommandLine.Options/tree/master.svg?style=svg)](https://circleci.com/gh/Siliconrob/CommandLine.Options/tree/master)

NDesk.Options.OptionSet is built upon a key/value table, where the
 key is a option format string and the value is a delegate that is 
 invoked when the format string is matched.

 Option format strings:
  Regex-like BNF Grammar: 
    name: .+
    type: [=:]
    sep: ( [^{}]+ | '{' .+ '}' )?
    aliases: ( name type sep ) ( '|' name type sep )*
 
 Each '|'-delimited name is an alias for the associated action.  If the
 format string ends in a '=', it has a required value.  If the format
 string ends in a ':', it has an optional value.  If neither '=' or ':'
 is present, no value is supported.  `=' or `:' need only be defined on one
 alias, but if they are provided on more than one they must be consistent.

 Each alias portion may also end with a "key/value separator", which is used
 to split option values if the option accepts > 1 value.  If not specified,
 it defaults to '=' and ':'.  If specified, it can be any character except
 '{' and '}' OR the *string* between '{' and '}'.  If no separator should be
 used (i.e. the separate values should be distinct arguments), then "{}"
 should be used as the separator.

 Options are extracted either from the current option by looking for
 the option name followed by an '=' or ':', or is taken from the
 following option IFF:
  - The current option does not contain a '=' or a ':'
  - The current option requires a value (i.e. not a Option type of ':')

 The `name' used in the option format string does NOT include any leading
 option indicator, such as '-', '--', or '/'.  All three of these are
 permitted/required on any named option.

 Option bundling is permitted so long as:
   - '-' is used to start the option group
   - all of the bundled options are a single character
   - at most one of the bundled options accepts a value, and the value
     provided starts from the next character to the end of the string.

 This allows specifying '-a -b -c' as '-abc', and specifying '-D name=value'
 as '-Dname=value'.

 Option processing is disabled by specifying "--".  All options after "--"
 are returned by OptionSet.Parse() unchanged and unprocessed.

 Unprocessed options are returned from OptionSet.Parse().

 Examples:
  int verbose = 0;
  OptionSet p = new OptionSet ()
    .Add ("v", v => ++verbose)
    .Add ("name=|value=", v => Console.WriteLine (v));
  p.Parse (new string[]{"-v", "--v", "/v", "-name=A", "/name", "B", "extra"});

 The above would parse the argument string array, and would invoke the
 lambda expression three times, setting `verbose' to 3 when complete.  
 It would also print out "A" and "B" to standard output.
 The returned array would contain the string "extra".

 C# 3.0 collection initializers are supported and encouraged:
  var p = new OptionSet () {
    { "h|?|help", v => ShowHelp () },
  };

 System.ComponentModel.TypeConverter is also supported, allowing the use of
 custom data types in the callback type; TypeConverter.ConvertFromString()
 is used to convert the value option to an instance of the specified
 type:

  var p = new OptionSet () {
    { "foo=", (Foo f) => Console.WriteLine (f.ToString ()) },
  };

 Random other tidbits:
  - Boolean options (those w/o '=' or ':' in the option format string)
    are explicitly enabled if they are followed with '+', and explicitly
    disabled if they are followed with '-':
      string a = null;
      var p = new OptionSet () {
        { "a", s => a = s },
      };
      p.Parse (new string[]{"-a"});    sets v != null
      p.Parse (new string[]{"-a+"});   sets v != null
      p.Parse (new string[]{"-a-"});   sets v == null

