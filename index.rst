..
  Content of technical report.

  See http://docs.lsst.codes/en/latest/development/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-report-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.


.. include:: colors.rst

.. topic:: Note

    These notes will be constantly modified are do not represent an official release of the SuperTask and pipe_base re-design. These are intended for people to get familiar with the current work and to have some sort of updated documentation for future uses as well.


Getting Started
===============

These instructions and notes are for the stack version 11.0+2 (lsst_apps 11.0+2)


CmdLineActivator
----------------



Setting up environment
^^^^^^^^^^^^^^^^^^^^^^

Need to clone from pipe_base-x_ , branch tickets/DM-3756

.. prompt:: bash

    setup -v pipe_tasks -t v11_0
    git clone -b tickets/DM-4509 https://github.com/lsst-dm/pipe_base-x.git
    setup -v -r pipe_base-x/. -t v11_0


Running ExampleCmdLineTask
--------------------------

The exampleCmdLineTask_ computed the mean and the std of a raw image. The command line executable is not in the ``bin\`` folder. To run the example (Using the stack, which calls the method ``parseandRun()``) do:

.. prompt:: bash

    python $PIPE_TASKS_DIR/examples/exampleCmdLineTask.py $OBS_TEST_DIR/data/input/ --id

And the output is:

.. code-block:: none

    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/exampleTask.py'
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/test/exampleTask.py'
    : input=/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input
    : calib=None
    : output=None
    CameraMapper: Loading registry registry from /Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input/registry.sqlite3
    exampleTask: Processing data ID {'filter': 'g', 'visit': 1}
    exampleTask.stats: clipped mean=1184.70; meanErr=0.02; stdDev=33.64; stdDevErr=1.04
    exampleTask: Processing data ID {'filter': 'g', 'visit': 2}
    exampleTask.stats: clipped mean=1228.79; meanErr=0.02; stdDev=34.19; stdDevErr=nan
    exampleTask: Processing data ID {'filter': 'r', 'visit': 3}
    exampleTask.stats: clipped mean=1433.76; meanErr=0.03; stdDev=37.36; stdDevErr=0.93

In the new *framework* using SuperTask and CmdLineActivator, we can run the same example by calling:

.. prompt:: bash

    cmdLineActivator exampleCmdLineTask --extras $OBS_TEST_DIR/data/input/ --id

And the output is:

.. code-block:: none

    lsst.pipe.base.examples.exampleCmdLineTask

    Classes inside module lsst.pipe.base.examples.exampleCmdLineTask :

    exampleCmdLineTask.ExampleCmdLineConfig
    exampleCmdLineTask.ExampleCmdLineTask

    exampleTask was initiated
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/exampleTask.py'
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/test/exampleTask.py'
    : input=/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input
    : calib=None
    : output=None
    CameraMapper: Loading registry registry from /Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input/registry.sqlite3
    exampleTask was initiated
    exampleTask: Processing data ID {'filter': 'g', 'visit': 1}
    exampleTask.stats: clipped mean=1184.70; meanErr=0.02; stdDev=33.64; stdDevErr=1.04
    exampleTask was initiated
    exampleTask: Processing data ID {'filter': 'g', 'visit': 2}
    exampleTask.stats: clipped mean=1228.79; meanErr=0.02; stdDev=34.19; stdDevErr=nan
    exampleTask was initiated
    exampleTask: Processing data ID {'filter': 'r', 'visit': 3}
    exampleTask.stats: clipped mean=1433.76; meanErr=0.03; stdDev=37.36; stdDevErr=0.93

The outputs are almost the same but few extra printouts. The difference in running the task is that we call **cmdLineActivator** to run the given Task  (which is now a SuperTask) and separate the arguments from the cmdLineActivator and the task with :blueit:`--extras` 

The difference between both exampleCmdLineTasks (to be backwards compatible) is:

.. code-block:: diff

    --- a/Users/Matias/Dropbox/lsst_dm/Task_Redesign/pipe_base-x/python/lsst/pipe/base/examples/exampleCmdLineTask.py
    +++ b/Users/Matias/LSST_EUPS/DarwinX86/pipe_tasks/11.0+1/python/lsst/pipe/tasks/exampleCmdLineTask.py
    @@ -26 +26 @@ import lsst.pipe.base as pipeBase
    -from lsst.pipe.tasks.exampleStatsTasks import ExampleSigmaClippedStatsTask
    +from .exampleStatsTasks import ExampleSigmaClippedStatsTask
    @@ -51 +51 @@ class ExampleCmdLineConfig(pexConfig.Config):
    -class ExampleCmdLineTask(pipeBase.SuperTask):
    +class ExampleCmdLineTask(pipeBase.CmdLineTask):
    @@ -109,3 +109 @@ class ExampleCmdLineTask(pipeBase.SuperTask):
    -        #pipeBase.CmdLineTask.__init__(self, *args, **kwargs)
    -        super(ExampleCmdLineTask, self).__init__(*args, **kwargs)
    -
    +        pipeBase.CmdLineTask.__init__(self, *args, **kwargs)
    @@ -115 +113 @@ class ExampleCmdLineTask(pipeBase.SuperTask):
    -    def execute(self, dataRef):
    +    def run(self, dataRef):

So basically CmdLineTask --> SuperTask and run --> execute (more to come on this...)

To list the available tasks seen by CmdLineActivator (which for now only looks in lsst.pipe.base.examples):

.. prompt:: bash

    cmdLineActivator --list_tasks

and the output is:

.. code-block:: none

    lsst.pipe.base.examples.ExampleStats.ExampleMeanTask
    lsst.pipe.base.examples.ExampleStats.ExampleStdTask
    lsst.pipe.base.examples.SuperExampleStats.AllStatTask
    lsst.pipe.base.examples.exampleCmdLineTask.ExampleCmdLineTask
    lsst.pipe.base.examples.supertesttask.SuperTestTask
    lsst.pipe.base.examples.supertesttask.Super2Task
    lsst.pipe.base.examples.test1task.Test1Task
    lsst.pipe.base.examples.test2task.Test2Task


All these tasks can be called as shown before,  for example the task **ExampleMeanTask** only computes the Mean of an image (for the sake of the examples), the code  (shortened) is shown below:

.. code-block:: py

    from __future__ import absolute_import, division, print_function
    import lsst.pex.config as pexConfig
    from lsst.afw.image import MaskU
    import lsst.afw.math as afwMath
    import lsst.pex.config as pexConfig
    import lsst.pipe.base as pipeBase
    import lsst.pipe.base.super_task as super_task
    from lsst.pipe.base.super_task import SuperTask

    class ExampleMeanConfig(pexConfig.Config):
        """!Configuration for ExampleSigmaClippedStatsTask
        """
        badMaskPlanes = pexConfig.ListField(
            dtype = str,
            doc = "Mask planes that, if set, the associated pixel should not be included in the coaddTempExp.",
            default = ("EDGE",),
        )
        numSigmaClip = pexConfig.Field(
            doc = "number of sigmas at which to clip data",
            dtype = float,
            default = 3.0,
        )
        numIter = pexConfig.Field(
            doc = "number of iterations of sigma clipping",
            dtype = int,
            default = 2,
        )


    @super_task.wrapclass(super_task.wraprun)
    class ExampleMeanTask(SuperTask):

        ConfigClass = ExampleMeanConfig
        _default_name = "exampleMean"

        def __init__(self, *args, **kwargs):

            super(ExampleMeanTask, self).__init__(*args, **kwargs)
            #basetask.Task.__init__(self, *args, **kwargs)

        def pre_run(self):
            self._badPixelMask = MaskU.getPlaneBitMask(self.config.badMaskPlanes)
            self._statsControl = afwMath.StatisticsControl()
            self._statsControl.setNumSigmaClip(self.config.numSigmaClip)
            self._statsControl.setNumIter(self.config.numIter)
            self._statsControl.setAndMask(self._badPixelMask)


        @pipeBase.timeMethod
        def execute(self, dataRef):

            calExp = dataRef.get("raw")
            maskedImage = calExp.getMaskedImage()
            return self.run(maskedImage)

        def run(self, maskedImage):

            statObj = afwMath.makeStatistics(maskedImage, afwMath.MEANCLIP | afwMath.STDEVCLIP | afwMath.ERRORS,
                self._statsControl)
            mean, meanErr = statObj.getResult(afwMath.MEANCLIP)
            self.log.info("clipped mean=%0.2f; meanErr=%0.2f" % (mean, meanErr))

            self.output= pipeBase.Struct(
                mean = mean,
                meanErr = meanErr,
            )
            return self.output


And you can run this (Super)Task with (only one visit in filter r):

.. prompt:: bash

    cmdLineActivator exampleMeanTask --extras $OBS_TEST_DIR/data/input/ --id --filter=r

And the output is:

.. code-block:: none

    lsst.pipe.base.examples.ExampleStats

    Classes inside module lsst.pipe.base.examples.ExampleStats :

    ExampleStats.ExampleMeanConfig
    ExampleStats.ExampleMeanTask
    ExampleStats.ExampleStdConfig
    ExampleStats.ExampleStdTask

    exampleMean was initiated
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/exampleMean.py'
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/test/exampleMean.py'
    : input=/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input
    : calib=None
    : output=None
    CameraMapper: Loading registry registry from /Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input/registry.sqlite3
    exampleMean was initiated
    exampleMean: clipped mean=1433.76; meanErr=0.03


Writing (Super)Tasks in new format 
----------------------------------

To be completed...

Writing WorkFlowTasks
---------------------

To be completed....


A workflow task based on the examples computes mean, std and mean again as separate tasks

.. code-block:: py

    from __future__ import absolute_import, division, print_function
    from lsst.pipe.base.workflow import WorkFlowSeqTask, WorkFlowParTask
    from lsst.pipe.base.examples.ExampleStats import ExampleMeanTask
    from lsst.pipe.base.examples.ExampleStats import ExampleStdTask
    import lsst.pex.config as pexConfig


    class AllStatConfig(pexConfig.Config):
        """
        Config
        """
        minval = pexConfig.Field(
            dtype=int,
            doc="Min value",
            default=2,
        )

    class AllStatTask(WorkFlowSeqTask):
        """
        SuperTest
        """
        ConfigClass = AllStatConfig
        _default_name = 'All_Stats'

        def __init__(self, config=None, name=None, parent_task=None, log=None, activator=None):
            super(AllStatTask, self).__init__(config, name, parent_task, log, activator)

            Mean = ExampleMeanTask()
            Mean.config.numSigmaClip = 4.0

            Mean2 = ExampleMeanTask(name='mean 2nd value')
            Mean2.config.numSigmaClip = 5.0

            Std = ExampleStdTask()

            self.link(Mean, Mean2, Std)

To run this WorkFlow (which is a SuperTask by itself):

.. prompt:: bash

    cmdLineActivator allstatTask --extras $OBS_TEST_DIR/data/input/ --id filter=r

And the output:

.. code-block:: none

    lsst.pipe.base.examples.SuperExampleStats

    Classes inside module lsst.pipe.base.examples.SuperExampleStats :

    SuperExampleStats.AllStatConfig
    SuperExampleStats.AllStatTask

    All_Stats was initiated
    exampleMean was initiated
    mean_2nd_value was initiated
    exampleStd was initiated
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/All_Stats.py'
    : Config override file does not exist: '/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/config/test/All_Stats.py'
    : input=/Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input
    : calib=None
    : output=None
    CameraMapper: Loading registry registry from /Users/Matias/LSST_EUPS/DarwinX86/obs_test/11.0+1/data/input/registry.sqlite3

    * Workflow Tree * :

     All_Stats
    |    +--> exampleMean
    |    +--> mean_2nd_value
    |    +--> exampleStd

    All_Stats was initiated
    exampleMean was initiated
    mean_2nd_value was initiated
    exampleStd was initiated
    I am running All_Stats Using cmdLine activator
    All_Stats: Processing data ID {'filter': 'r', 'visit': 3}
    exampleMean: clipped mean=1433.89; meanErr=0.03
    mean 2nd value: clipped mean=1433.99; meanErr=0.03
    exampleStd: stdDev=37.36; stdDevErr=0.93

which produce the following dot file (after rendered):

.. figure:: /_static/figures/All_Stats.png
 :name: AllStatTask
 :alt: WorkFlow for AllStatTask

 : Rendering version of the dot file generated in the process


.. _pipe_base-x: https://github.com/lsst-dm/pipe_base-x/tree/tickets/DM-3756
.. _exampleCmdLineTask: https://github.com/lsst/pipe_tasks/blob/master/python/lsst/pipe/tasks/exampleCmdLineTask.py
