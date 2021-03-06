# Using Generators in Gall
Throughout the basic course, we have mostly typed pokes in the form:
```
:chanel &chanel-action [%increase-counter step=9]
```
Gall has support for routing input to generators before passing it to the agent, and we can use this to make the above agent interactions cleaner. This particularly helps for agents that users will access via the Dojo.

## Gall Generator Mechanics
Let's take an example with the `s3-store` agent. We can poke it like so:
```
:s3-store|set-endpoint 'myendpoint.com'
```

The above expands to "poke `%s3-store` with the output from calling `+s3-store/set-endpoint 'myendpoint.com'`." This is similar to how marks look for marks like `chanel-action` in `mar/chanel/action.hoon`.

The output is passed to `on-poke` of `s3-store` as `[mark data]`.

### Example: `gen/s3-store/set-endpoint.hoon`
Open the file `gen/s3-store/set-endpoint.hoon`. Its code (as of this writing) is:
```
/-  *s3
:-  %say
|=  $:  [now=@da eny=@uvJ =beak]
        [[endpoint=@t ~] ~]
    ==
:-  %s3-action
^-  action
[%set-endpoint endpoint]
```
The arguments to this `%say` generator are:
1. `[now=@da eny=@uvJ =beak]`, the normal environment variables passed to a `%say` generator.
2. `[endpoint=@t ~]`, a list of arguments
3. `~`: this generator takes no optional arguments

In our example of `:s3-store|set-endpoint 'myendpoint.com'`, the generator takes `'myendpoint.com'` as the first argument, and evaluates it to `[%s3-action [%set-endpoint endpoint]]`.

We can check this at the Dojo by running the generator directly:
```
> +s3-store/set-endpoint 'myendpoint.com'
[%set-endpoint endpoint='myendpoint.com']
```
Notice that the Dojo automatically applies the mark to the data and removes the mark: this is a normal `%say` generator; no magic.

## Raw `|` Commands
In the dojo, any command like `|reset` or `|commit` is simply a shortcut for `:hood|reset` or `:hood|commit`. The `hood` generator is run, and then the `hood` agent processes the result.

To see the commands available to `hood` and the variables for those commands, just browse the `gen/hood` directory.  Each generator file represents a command, and you can open the files to see the exact sample that each command takes.

### Example: `|commit`
When we run `|commit %home`, that calls the generator `gen/hood/commit.hoon` with argument `%home`. It returns a cell like so:
```
:- %kiln-commit
[mon auto]
```
So a `%home` argument returns `[%kiln-commit [%home %.n]]`. We can check this in the Dojo:
```
> +hood/commit %home
[%home %.n]
```
If run as `|commit`, the whole `[%kiln-commit [%home %.n]]` is passed as `[mark vase]` to the `on-poke` of `hood`, from whence it is passed to [lib/hood/kiln.hoon](https://github.com/urbit/urbit/blob/35946bd168734c6d16adb61b04c314362cdf344d/pkg/arvo/lib/hood/kiln.hoon#L420). There, the mark is matched and handled.

## Summary
    You can make these simple generators for your own applications quite easily. Just follow the approach of `s3-store` above, and you can make separate `|` commands for all the actions you want your users to perform from the Dojo.
