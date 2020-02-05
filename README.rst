Introduction
------------
This is the robot scripts for OLRS.

Directory Structure
-------------------

GIT Branch Guidelines
---------------------
For the automation scripts we will have a main "common" branch. The "common" branch is the branch which contains working scripts and the branch which scripts should be run with when producing "official" results.

Off of the common branch will be "feature" branches. "Feature" branches will be used for script development. Almost all script development should be first applied to a feature branch.

"Feature" branches should be named such that reading the branch name one should understand what work that branch contains. Examples of good names include ca-ohv-transaction-robot which tells us that this is state specific (CA), it is the OHV Transaction, and it is the robot scripts helpful as these branches are seen by the whole team. When choosing a branch name make it distiinguishable from other names. A bad example of a branch name when branching from ca-logbook-robot would be my-ca-logbook-robot. It is a bad name because it looks almost exactly the same, it doesn't tell us who "me" is, and it doesn't describe how this new branch is different then ca-logbook-robot. Instead I should name it logbook-sort-by-vin-emanlove. Note the script developers names aren't required in the branch name. It is just "my" is not descriptive and actually adding the developers name to the branch may make it too long.

When merging a "feature" branch back into the main "common" branch perform a non-fast-forward merge. This allows for one to go back into the history a see a distinctive branch of work. Fast-forwarding puts commits inline and for working with feature branches it loses some helpful information. To perform a non-fast-foward merge using the --no-ff argument, as in::

    git merge --no-ff logbook-sort-by-vin-robot

Script Guidelines
-----------------
Test Suites
~~~~~~~~~~~
- Test suites should be named with the ```.robot``` extension.

Keywords
~~~~~~~~
- Keyword Names Should Be CamelCase With Spaces Between Words. \[**Correct**\] Don'tTitleThemWithoutSpace \[**Incorrect**\] nor Should_they_be_spaced_with_underscores \[**Incorrect**\]
- Keyword names should describe what they do
- Keywords should focus on doing one thing. They should not try to do multiple "actions" or "steps" at once. So something like ```Log Into Application And Start A Transaction And Enter Data``` should be broken down into, in this example, three different keywords: ```Log Into Application```, ```Start A Transaction```, and ```Enter Data```.
  
Validation Keywords vs. "Action" Keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Action keywords should be seperate from Validation Keywords. By Validation Keywords we mean keywords that are verifing and reporting on a checkpoint.
- Validation keywords should provide a clear and relevant error message if the validation failed. They should not rely on default error messages. For example "2 is not equal to 5" doesn't explain the failure as well as "Expecting 5 deals in batch but batch only contained 2 deals."

Test Cases
~~~~~~~~~~
- Test case names should describe what they do
- Test case names **should not** contain test case numbers

When to Log and at what level
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Don't use WARN for debugging. Use DEBUG.

Filenames and Directories
~~~~~~~~~~~~~~~~~~~~~~~~~
ToDo

Rename and Depreciate - Don't Just ReWork
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
When scripting if you modify an existing keyword in either the functionality or the argument structure don't just modify the existing keyword. Instead you showed create a new keyword and possibly depreciate the old

Here is an example of an existing keyword

.. code::  robotframework
	   
    Validate Sorted Fields Are In Ascending/Descending Order
       [Arguments]  ${Status_List}  ${Order}
       [Documentation]  This keyword is Used to Sort the Fields In Descending Order
       ${Temp_List}  Copy List  ${Status_List}
       Sort List  ${Status_List}
       Run Keyword If  '${Order}'=='Desc'  Reverse List  ${Status_List}
       Should Be Equal  ${Temp_List}  ${Status_List}
	   

that was then modified by another developer

.. code::  robotframework
	   
    Validate Sorted Fields Are In Ascending/Descending Order
       [Arguments]  ${Status_List}  ${Order}
       [Documentation]  This keyword is Used to Sort the Fields In Descending Order
       Remove Values From List  ${Status_List}  ${EMPTY}
       Remove Values From List  ${Status_List}  < View Transaction >
       ${Temp_List}  Copy List  ${Status_List}
       Sort List  ${Status_List}
       Log List  ${Status_List}
       Run Keyword If  '${Order}'=='Desc'  Reverse List  ${Status_List}
       Should Be Equal  ${Temp_List}  ${Status_List}

\[**Incorrect**\]

Instead, with the added remove "${EMPTY}" and "< View Transaction >" list items, this modified keyowrd should have been renamed. For example,

.. code::  robotframework
	   
    Validate Sorted Fields Are In Ascending/Descending Order Ignoring Empty And View Transaction
       [Arguments]  ${Status_List}  ${Order}
       [Documentation]  This keyword is Used to Sort the Fields In Descending Order and ignoring "Empty" And "View Transaction" list items
       Remove Values From List  ${Status_List}  ${EMPTY}
       Remove Values From List  ${Status_List}  < View Transaction >
       ...

Now it is important to also consider whether or not a new keyword is actually needed. In the above example it is probably best to simple remove those items from the list beforehand and then call the existing keyword. This would prevent keyword bloat. Consider another example where a developer adds an extra argument to an existing keyword. Here we have an existing keyword for selecting make 

.. code::  robotframework
	   
    Select Make With Vehicle Widget
       [Arguments]  ${Description}
       [Documentation]  This keyword selects the vehicle make withn the vehicle widget
       ...   using the given value ${Description}
       # ... do something ...

Due to changes with the widget the widget needs both the description and coded value but this change will be phased in over time. We know we want in the end have one keyword to select the make named "Select Make With Vehicle Widget". The wrong way to introduce this change is to simply change the keyword and add the argument

.. code::  robotframework
	   
    Select Make With Vehicle Widget
       [Arguments]  ${Description}  ${coded_value}
       [Documentation]  This keyword selects the vehicle make withn the vehicle widget
       ...   using the given value ${Description}
       # ... do something ...


\[**Incorrect**\]

Instead here we will use a temporary new keyword

.. code::  robotframework
	   
    Select Make With Vehicle Widget Using Description and Coded Value
       [Arguments]  ${Description}  ${coded_value}
       [Documentation]  This keyword selects the vehicle make withn the vehicle widget
       ...   using the given values: ${Description} and ${coded_value}
       # ... do something ...

and deprecate the old keyword. There are several ways to do this but iusing the built in functionality we would give notice to users that the old keyword is deprecated and to use the use new keyword.

.. code::  robotframework
	   
    Select Make With Vehicle Widget
       [Arguments]  ${Description}
       [Documentation]  \*DEPRECATED. Please start using 'Select Make With Vehicle Widget Using Description and Coded Value' instead.\*
       ...   This keyword selects the vehicle make withn the vehicle widget
       ...   using the given value ${Description}
       # ... do something ...

See http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#deprecating-keywords for more information about using the built-in deprecate keyword functionality

Style Guide
~~~~~~~~~~~
- Use Spaces and **NOT TABS** for spacing.
- Use four spaces as the Robot Framework separator. (This is to ease the differences between RIDE and other text editors and reduce the number of white-space differences.)
- Use ${/} for directory seperators within paths. **DO NOT** use // nor / nor \.
- Use proper capitalization for objects in path. Some OSes are case sensitive so if the actual path has both upper and lower case letters match the path name exactly including capitalization.
  
Script Documentation
~~~~~~~~~~~~~~~~~~~~
- Keywords should have documentation setting. Example:

.. code::  robotframework

    *** Keywords ***
    Add Note To Deal
        [Documentation] This keyword adds a notation to the deal within
        the deal summary shown in logbook.


Documentation
~~~~~~~~~~~~~
- All documentation shall use the ReST format.

Feel free to contibute to this document. If we have talked about and agreed upon some guideline but it is not written down here please add it or ask for it to be added.

Tagging Stratergy
~~~~~~~~~~~~~~~~~
ToDo
