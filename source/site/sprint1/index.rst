Sprint 1
========

Contents:

.. toctree::
  :maxdepth: 2

  
User Story
----------

An initial site is required to begin the process of creating content.

Detail
^^^^^^

The core requirement for this story is the existence of a website that can be publically hosted
and become the basis for an improvement process.

At this stage the technologies used are largely irrelevant, it is expected that a more fitting solution will be found as the project progresses.

Implementation details
^^^^^^^^^^^^^^^^^^^^^^

The initial plan is for documents to be stored on Github and copied into a website as they're updated. However at the time of coding the website I couldn't determine how to perform this update without querying Github with each request. For this reason I decided it would be beneficial to completely separate the data from the website and store it in a backend database.

Metadata for the pages is stored as YAML at the top of each page. These values will become columns in the database.

It was decided to use Azure Storage Tables for storage because of the negligible cost and simple deployment. A small console application was created to download the latest commit from Github and push it into Azure Storage Tables. An incrementing version number was used for the partition key, and the URI slug for the row key. Currently this leaves previous version behind, in a future releases these will either be accessible through the website (to view previous versions of pages) or regularly purged.

On the frontend the website simply queries the table for the most recent version number, downloads the entire collection of documents, creates index links and extracts the required page. A more efficient will be required.

Issues
^^^^^^

There are currently a number of outstanding issues with this release.

1. The table of contents is currently an unsorted list of entries. These need to be split into groups with a display order added.
2. The coding standard for the website is poor, in the very near future it will need a rewrite by someone with coding skills.
