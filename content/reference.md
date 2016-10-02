## Reference
Please note that there are a number of undocumented options. For more
information on these options, please view the source at:
[https://github.com/purpleidea/mgmt/](https://github.com/purpleidea/mgmt/).
If you feel that a well used option needs documenting here, please patch it!

### Overview of reference
* [Meta parameters](#meta-parameters): List of available resource meta parameters.
* [Graph definition file](#graph-definition-file): Main graph definition file.
* [Command line](#command-line): Command line parameters.

### Meta parameters
These meta parameters are special parameters (or properties) which can apply to
any resource. The usefulness of doing so will depend on the particular meta
parameter and resource combination.

#### AutoEdge
Boolean. Should we generate auto edges for this resource?

#### AutoGroup
Boolean. Should we attempt to automatically group this resource with others?

#### Noop
Boolean. Should the Apply portion of the CheckApply method of the resource
make any changes? Noop is a concatenation of no-operation.

#### Retry
Integer. The number of times to retry running the resource on error. Use -1 for
infinite. This currently applies for both the Watch operation (which can fail)
and for the CheckApply operation. While they could have separate values, I've
decided to use the same ones for both until there's a proper reason to want to
do something differently for the Watch errors.

#### Delay
Integer. Number of milliseconds to wait between retries. The same value is
shared between the Watch and CheckApply retries. This currently applies for both
the Watch operation (which can fail) and for the CheckApply operation. While
they could have separate values, I've decided to use the same ones for both
until there's a proper reason to want to do something differently for the Watch
errors.

### Graph definition file
graph.yaml is the compiled graph definition file. The format is currently
undocumented, but by looking through the [examples/](https://github.com/purpleidea/mgmt/tree/master/examples)
you can probably figure out most of it, as it's fairly intuitive.
