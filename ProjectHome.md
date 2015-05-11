The open-source **JCombinatorial library** facilitates parameterized testing with JUnit in an efficient manner. It consists of a series of parameter factories, each of which suits a different purpose.

Each factory takes a set of possible values for each test parameter and creates a list of value arrays, each of which represents the parameter values to be used for a single test-case of a parameterized JUnit test. The factories are as follows:

  * AllValuesParameterFactory, which generates parameters for the minimum number of test-cases required to test all given values of each parameter. (This is the most efficient factory and is recommended for smoke tests.)

  * AllPairsParameterFactory, which generates parameters for the minimum number of test-cases required to test all pairings of the given parameter values. (This factory provides what is typically seen as a good balance between efficiency and completeness in testing. You can read more about this "all-pairs" or "pairwise" approach in [an article by the United States Computer Security Resource Center](http://csrc.nist.gov/groups/SNS/acts/index.html) or in [the Wikipedia article](http://en.wikipedia.org/wiki/All-pairs_testing).)

  * AllCombinationsParameterFactory, which generates all possible combinations of the given parameter values. (This exhaustive factory is recommended when the time required to execute each test-case is very short and the consequence of leaving a bug undiscovered would be quite severe.)


---

This library was developed by Jeremy Reeder at [Bodybuilding.com](http://www.bodybuilding.com/) and is subject to the terms of the New BSD License. Copyright © MMXI, Bodybuilding.com LLC. Portions copyright © MMX Ng Pan Wei. ([JWise library](http://code.google.com/p/jwise/)) Further details of the license agreements can be found within the downloadable package.