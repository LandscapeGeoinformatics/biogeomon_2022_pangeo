FAIR and Reproducible Research
------------------------------

.. figure:: https://www.openaire.eu/images/Guides/EC_FAIR_data.png
   :height: 300

   https://www.openaire.eu/how-to-make-your-data-fair

Check out the following article in the Journal Scientific Data:

  Wilkinson, M., Dumontier, M., Aalbersberg, I. et al. The FAIR Guiding Principles for scientific data management and stewardship. Sci Data 3, 160018 (2016). https://doi.org/10.1038/sdata.2016.18

  https://www.nature.com/articles/sdata201618




Use basic Git and GitHub
~~~~~~~~~~~~~~~~~~~~~~~~


Version control is better than mailing files back and forth:

- Nothing that is committed to version control is ever lost, unless you work really, really hard at it. Since all old versions of files are saved, it’s always possible to go back in time to see exactly who wrote what on a particular day, or what version of a program was used to generate a particular set of results.

- As we have this record of who made what changes when, we know who to ask if we have questions later on, and, if needed, revert to a previous version, much like the “undo” feature in an editor.

- When several people collaborate in the same project, it’s possible to accidentally overlook or overwrite someone’s changes. The version control system automatically notifies users whenever there’s a conflict between one person’s work and another’s.

Teams are not the only ones to benefit from version control: lone researchers can benefit immensely. Keeping a record of what was changed, when, and why is extremely useful for all researchers if they ever need to come back to the project later on (e.g., a year later, when memory has faded).

Version control is the lab notebook of the digital world: it’s what professionals use to keep track of what they’ve done and to collaborate with other people. Every large software development project relies on it, and most programmers use it for their small jobs as well. And it isn’t just for software: books, papers, small data sets, and anything that changes over time or needs to be shared can and should be stored in a version control system.

1. `Git <https://git-scm.com/>`__ is a `version control <https://en.wikipedia.org/wiki/Version_control>`__ system.

2. `GitHub <https://github.com/>`__ is one of several online platform for public and distributed Git repository access and sharing. Others are `Bitbucket <https://bitbucket.org/>`__ or `Gitlab <https://gitlab.com/>`__ and large organizations often have their own instances as well (e.g. Uni Tartu has an own Gitlab).

3. Typically, you either start a) completely fresh from the start, b) with a local folder that you want to put under version control and share, or c) with an existing online GitHub repository, that you want to update or contribute to.


a) Fresh start
~~~~~~~~~~~~~~

Go on Github (or your preferred Git platform), and create a new repository. On Github or Gitlab it will tell you the exact steps to initial a fresh repository or link a local folder.

Something along those lines:

  The repository for this project is empty

  You can get started by cloning the repository or start adding files to it with one of the following options.

**Command line instructions**

You can also upload existing files from your computer using the instructions below.
Git global setup

.. code:: bash

   git config --global user.name "Alexander Kmoch"
   git config --global user.email "alexander.kmoch@ut.ee"


**Clone** the new empty repository


.. code:: bash

   git clone git@github.com:allixender/test.git
   cd test12
   git switch -c main
   touch README.md
   git add README.md
   git commit -m "add README"
   git push -u origin main


**Push** an existing folder into version control

.. code:: bash

   cd existing_folder
   git init --initial-branch=main
   git remote add origin git@github.com:allixender/test.git
   git add .
   git commit -m "Initial commit"
   git push -u origin main

**OR** Push and link an existing Git repository

.. code:: bash

   cd existing_repo
   git remote rename origin old-origin
   git remote add origin git@github.com:allixender/test.git
   git push -u origin --all
   git push -u origin --tags



b) Local folder version control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typically, you can start immediately and initialize Git version control with no further hassle:

.. code:: bash

   git init .

And from hence you will do the basic workflow as follows, over and over:

.. code:: bash

   git status


You can use/create the **.gitignore** file to ignore temporary or non-relevant files or prevent accidentally adding sensitive files to the version control.

The most basic workflow is checking with "git status" as seen above and selectively adding updated or new files to the commit process (staging) ...

.. code:: bash

   git add <file>

   # or

   git add .


The you commit (like checking in for good, register for history). Provide a message, for yourself or others, this will be related to that particular commit.

.. code:: bash

   git commit -m "commit message for yourself or others"


If your local repository folder is already linked to an online Git repository, like GitHub, you can "push" the latest commits (aka) changes to the online version, where they are "safe" and can be seen by others, and yourself, in case you delete your local folder or want to continue working on another computer.
The exact command depends on your initial configuration, but don't worry, lots of practical decisions are almost automatic.

.. code:: bash

   git push origin

   # git push origin main
   # git push origin master
   # ... you will see, typically it's done once and you can just do "git push origin"


c) Existing online GitHub repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have an own existing GitHub repository online, you would typically "clone" it into a local working directory.

.. code:: bash

    git clone <your online GitHub repository URL>


Once you start relating between your local Git folder, your GitHub repository (in your account) and other online GitHub repositories, you need a few more things.

If you have changes in your upstream (origin, e.g. GitHub) that you know of and you need to update your local Git folder:

.. code:: bash

   git pull origin
   # git pull origin main
   # git pull origin master
   # ... you will see, typically it's done once and you can just do "git pull origin"


If you want to contribute to someone else's GitHub repository, **don't** immediately clone their repo into your local folder. Rather, create a **fork** online in GitHub, this will add a linked copy into your account.
Then "clone" that repository from your account into your local working folder. Once you have "pushed" your local changes back online to your online GitHub repository, you can create a *pull request* to the other person's original repository.


If you by any chance run into the situation that you have online and local accidentally changed the same file, you will run into a "merge conflict", then "pull" will not be allowed by Git. Instead you will have "fetch" the changes and then merge (just really local manual deciding and updating) and "commit" and "push" the merged files again.

.. code:: bash

   # assuming your default working branch is main (online aka origin)
   # git fetch loads the online version without applying the changes
   git fetch origin main
   # with git merge you tell Git to apply changes from the origin/main branch to your local (main) working branch
   git merge origin/main
   # this can cause the message "merge conflict" and then tell you which files you'll have to "repair" or decide which is the more important one
   # ... you editing the files
   git add <the repaired target files>
   git commit -m "fixed merge conflict"
   git push origin main



Git Cheat Sheet for your convenience
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- https://www.jrebel.com/blog/git-cheat-sheet

- https://www.jrebel.com/system/files/git-cheat-sheet.pdf


ZENODO
~~~~~~

  Know how to deposit code and data in a indexed scientific repository with a DOI


.. figure:: https://about.zenodo.org/static/img/logos/zenodo-gradient-round.svg
   :height: 100

   https://zenodo.org/

Zenodo assigns all publicly available uploads a Digital Object Identifier (DOI) to make the upload easily and uniquely citeable. Zenodo further supports harvesting of all content via an open metadata sharing protocol.

- (All) Research Shared, your one stop research shop!

- Discoverable, be found!

- create your own repository

- more than just a drop box! Your research output is stored safely for the future in same cloud infrastructure as research data from CERN's Large Hadron Collider and using CERN's battle-tested repository software Invenio, which is used by some of the world's largest repositories such as INSPIRE HEP and CERN Document Server.

- Reporting - tell your funding agency! Zenodo is integrated into reporting lines for research funded by the European Commission via OpenAIRE.




Reproducibility guidelines
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://reproducible-agile.github.io/public/images/reproducible-AGILE-logo-square.svg
   :height: 100

   https://reproducible-agile.github.io/

https://en.wikipedia.org/wiki/Reproducibility#Reproducible_research

Reproducible research is a crucial topic for any research domain using computational processes, including GIScience and EarthSciences. Recent research has shown, that many publications leave room for improvement regarding their computational reproducibility.  The AGILE council generously supports an initiative for development of new guidelines for AGILE conference submissions to improve reproducibility. The planned duration of the initiative is January 2019 to June 2019. The initiative will prepare new author guidelines and reviewer guidelines suitable for all submission types (full, short, poster).

`AGILE Reproducible Paper Guidelines - December 2020.pdf <https://osf.io/numa5>`__


More skills - The Carpentries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://carpentries.org/assets/img/TheCarpentries.svg
   :height: 100

   https://carpentries.org/


Since 1998, **Software Carpentry** has been teaching researchers the computing skills they need to get more done in less time and with less pain. Our volunteer instructors have run hundreds of events for more than 34,000 researchers since 2012. All of our lesson materials are freely reusable under the **Creative Commons - Attribution license**.

The **Software Carpentry** Foundation and its sibling lesson project, **Data Carpentry**, have merged to become **The Carpentries**, a fiscally sponsored project of Community Initiatives, a 501(c)3 non-profit incorporated in the United States. See the staff page for The Carpentries.

- https://swcarpentry.github.io/git-novice/



Example: MyBinder online notebook demo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Stitching it all together:

- this workshop has a GitHub repository, with the notebook and the *conda* / *Python* environment under version control
- the Binder software on `MyBinder <https://mybinder.org/>`__ can build an executable container from the repository
- we can deposit and archive the whole workshop materials self-contained including the reproducible notebook workflow with a DOI on Zenodo, make it citeable
- the DOI can be cited and is indexed in OpenAIRE and can be used in project/science reporting and other scientific works


.. image:: https://mybinder.org/badge_logo.svg
   :target: https://mybinder.org/v2/gh/LandscapeGeoinformatics/biogeomon_2022_pangeo/HEAD?labpath=notebook%2Fworkshop.ipynb


Spatio-temporal trend analysis of spatial climate data (temperature and
rainfall) using Python (2021) Alexander Kmoch, Bruno Montibeller, Holger
Virro, Evelyn Uuemaa, |DOI|


.. |DOI| image:: https://zenodo.org/badge/DOI/10.5281/zenodo.5876348.svg
   :target: https://doi.org/10.5281/zenodo.5876348
