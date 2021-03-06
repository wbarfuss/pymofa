
python modeling framework (pyMoFa)
==================================
is a collection of simple functions to run and evaluate computer models
systematically.

.. contents::

Usage
-----
To use the modeling framework write a python script representing one
"experiment" you want to perform with your model.

There you have to specify the path where you want to save your data, the sample
size you want to use, the parameter combinations with which you want to run
your model and a function to set up and run your model and save resulting data
::
    # File: Experiment.py

    # Imports
    from ModelingFramwork import experiment_handling as eh
    import YourModel
    import cPickle
    import itertools as it
    import otherstuff

    # For use on the cluster
    SAVE_PATH = "path/where/to/store/the/ExperimentData"

    def RUN_FUNC(pa, ra, me, ters, filename):
       """
       <Your Docs>

       Parameters
       ----------
       ...
       filename: string
           the filename where to pickle - passed from "experiment_handling"
           This needs to be the last parameter of the RUN_FUNC
           Experiment handling makes sure that each model run gets an unique ID
       """
       # poss. do something with the parameters given above

       # Running the model and obtian what you want to save
       m = YourModel(mo, del, pa, ra, me, ters)
       m.run()
       result = m.get_result()

       # Save Result
       cPickle.dump(result, open(filename, "wb"))

       # Determine some exit status and retun
       # if exit_status != 1 MoFa will print a note
       # of course, the data saving can be also dependend on exit status
       exit_status = <determine exit status>
       return exit_status

    # Parameter combinations
    pa = [1, 2, 3, 4]
    ra = [1, 3, 5, 7, 9]
    me = [0, 1]
    ters = [42]
    PARAM_COMBS = list(it.product(pa, ra, me, ters))

    # Sample Size
    SAMPLE_SIZE = 100

    # Now start the computation
    eh.compute(RUN_FUNC, PARAM_COMBS, SAMPLE_SIZE, SAVE_PATH)


Additionally you can post-process the generated model data for evaluation. For
this you need to provide an index, i.e. the subset of parameters that are the
parameters you want to study in more detail, evaluation functions and a name of
the resaved data file.
::
    # Continuation of Experiment.py

    # index is a dict with int keys giving the position in RUN_FUN parameter
    # list and string values giving their names
    INDEX = {0: "pa", 1: "ra"}

    # eva is a dict of evaluation functions computing a "macro" quantity from
    # the "micro" data, stored by RUN_FUN. These function need to receive an
    # interable of filenames as their parameter. The corresponding files were
    # generated by RUN_FUNC and need tobe treated accordingly. The keys of the
    # eva dict are the corresponding names of the "macro" quantity.
    EVA = {"<macroQ1>": lambda fnames: <do_something_with>([np.load(f)
                                                            for f in fnames]),
          "<macroQ2>": lambda fnames: <do_something_different>([np.load(f)
                                                                for f
                                                                in fnames])}
    # the name of the post processed data
    NAME = "PostProcessedData_me0.pkl"

    # another set of parameter combinations (optional) - you could also use
    # PARAM_COMBS from above
    PARAM_COMBS_me0 = list(it.product(pa, ra, [0], ters))

    # This will now check for the largest available sample size and use it
    eh.resave_data(SAVE_PATH, PARAM_COMBS_me0, INDEX, EVA, NAME)
    # The file gets automatically saved at the sub-folder "./data/" which needs
    # to exist

For further documentation, use the source!

Tests
-----
using `pytest <http://docs.pytest.org/en/latest/>`_ with
`pylama <https://github.com/klen/pylama#pytest-integration>`_
(including `pylama-pylint <https://github.com/klen/pylama_pylint>`_)
and test coverage reports with the `pytest` plugin
`pytest-cov <https://github.com/pytest-dev/pytest-cov>`_.

To be installed with::

    $> pip install pytest pylama pylama-pylint pytest-cov
    
The config file is <pytest.ini>.
    
Write tests and make sure that they pass by::

    $> py.test

