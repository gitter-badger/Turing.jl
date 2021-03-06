How the Compiler Works
=========

In general, a compiler translates one programming language into another. Here the compiler in Turing translates probabilistic programs into normal Julia programs by transforming the three probabilistic operations into normal Julia statements. This transformation is achieved by using macros in Julia.

The ``@assume`` macro
---------

Using::

  @assume m ~ Normal(0, 1; static=true)


as an example, the macro ``@assume`` will

* Check if there is any additional argument ``static`` or ``param``. If so, it will do settings regarding to the argument, and discard this argument inside the distribution expression, i.e. turning ``Normal(0, 1; static=true)`` into ``Normal(0, 1)``.
* The ``Distribution`` type will be converted to a custom wrapper called ``dDistribution`` (differentiable distribution) by appending a letter 'd' to the corresponding argument in the expression, i.e. turning ``Normal(0, 1)`` to ``dNormal(0, 1)``, whenever the corresponding differentiable distribution is supported by Turing.
* The macro will return an expression which calls a function named ``assume()`` by passing through the identity of this macro. The identity is generated by ``gensym()`` and will be used to replay priors in HMC.

After all of these three steps, the Julia statement to be exectued will be::

  m = Turing.assume(Turing.sampler, dNormal(0, 1), Prior(Symbol($(string(sym)))))

The aim of using types ``Prior`` is to pass an identity of each ``@assume`` macro, which is corresponding to each prior, by which priors can be replayed by the HMC sampler respectively.

  The priors passed to the sampler will be stored in a data structure named ``PriorContainer``, which holds a dictionary with each ``Prior`` as its key and corresponding value being a list whose tail is connected with its head. This way of prior replay supports any abritary data structure to be used in a Turing program.

The ``@observe`` macro
---------

Exactly similarly to``@assume``, ``@observe`` will firstly do settings according to additional arguments, discard the annotation from distribution construction and then turn ``Distribution`` into ``dDistribution``. The difference happens in the third step, where ``@observe`` will pass the concrete value of the variable observed to a function called ``@observe``.

For instance,::

  @observe xs[1] ~ Normal(0, 1)

with::

  xs = [1.5, 2.0]

will become::

  Turing.assume(Turing.sampler, dNormal(0, 1), 1.5)

The ``@predict`` macro
---------

Using::

  @predict m

as an example, if the value of ``m`` is ``1.1`` at the time when this macro is called, the statement above will become::

  Turing.predict(Turing.sampler, :m, 1.1)


The ``@model`` macro
---------

The macro ``@model`` wraps all the original and generated statements in its scope into an expression ``ex``, stores this expression in the Turing's global scope as::

  TURING[:modelex] = ex

and returns this expression.
