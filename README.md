# vis-editorconfig

A [vis][vis] plugin for [editorconfig][ec].

[vis]: https://github.com/martanne/vis
[ec]: http://editorconfig.org/

## Installation

You'll need the Lua wrapper for editorconfig-core installed. This can
be done through luarocks: `luarocks install editorconfig-core`

```shell
git clone https://github.com/vktec/vis-editorconfig "$HOME/.config/vis/editorconfig"
```

Then add `require('editorconfig/edconf')` to your `visrc.lua`.

## Functionality

Not all editorconfig functionality is supported by vis and hence by this
plugin. At this moment, there is full support for the following settings:

- indent_style
- indent_size
- tab_width
- insert_final_newline

The following settings are implemented partially and / or support is
turned off by default:

- trim_trailing_whitespace: Turned off by default, can be enabled
  via `:set edconfhooks on`.
- end_of_line: Turned off by default, partial support can be enabled
  via `:set edconfhooks on`. Only `crlf` and `lf` are supported, plain
  `cr` is not. The implementation is also very basic. If end_of_line
  is set to `crlf`, a return character will be inserted at the end of
  each line that does not yet end with `crlf`. If end_of_line is set
  to `lf`, return characters at the end of a line will be stripped.
  While I would encourage every vis user to stick to `lf` terminated
  files, this might be convenient if, for some reason, they do need
  to compose `crlf` terminated files.
- max_line_length: Turned off by default, partial support can be
  enabled via `:set edconfhooks on`. There is no straightforward way
  to automatically wrap content that might be written in arbitrary
  programming (or non-programming) languages. For that reason,
  vis-editorconfig doesn't even try. If max_line_length is enabled,
  vis-editorconfig issues a warning, however, indicating which lines
  are longer than the specified length. Note that you will miss this
  warning if you write your file with `:wq`, so if you care about it,
  write it with `:w<RETURN>:q`.

The reason those last three settings are optional and turned off by
default is their scalability. Each of those operations is implemented
as a pre-save-hook with a complexity of O(n), where n is the filesize.
Since vis is incredibly good at editing huge files efficiently, there
seems to be a very real danger that those hooks could cause the editor
to freeze just before a user's valuable changes are written to disk.

You can turn support for those pre-save-hooks on or off at any time
by running

    :set edconfhooks on

or

    :set edconfhooks off

If `edconfhooks` are enabled, they will be executed as configured in
`.editorconfig`. If you want to take a less cautious approach and enable
these hooks by default, simply add an additional line below the module
import in `visrc.lua`:

    require('editorconfig/edconf')
    vis:command('set edconfhooks on')   -- supposing you did previously
                                        -- require('vis')

### Setting arbitrary options

To some it might be convenient to use editorconfig files to set arbitrary
vis options. Therefore, the vis editorconfig plugin uses the `vis_`
prefix in keys as an interface to anything that can be specified with
`:set`.

Instead of running `:set spelllang en_US` after opening a file, for
instance, one can also add

    vis_spelllang = en_US

to the appropriate section of an editorconfig file.

Note that keys in editorconfig are case-insensitive. The specification
suggests to lowercase them. Therefore, it is only possible to use this
mechanism for lowercase options. If you really need to use upper or
camel case options, add an alias to your `visrc.lua`:

```lua
vis:option_register("snake_case_option", "string", function(value)
  vis:command("set camelCaseOption " .. value)
end, "Alias for camelCaseOption")
```
