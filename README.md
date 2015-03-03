calibre-book-export
===================

## Intelligently export books from Calibre based on flags in the DB.

Calibre Book Export is a wrapper around the "calibredb export" that
intelligently manages an exported collection of books for using a tool
like Rsync, Unison, or FolderSync on Android (this author's current
toolset) to copy the collection elsewhere.

## Key Features

- Maintains an export directory based on tags or custom column data in
  the Calibre database.
  
- Does not change timestamps or other filesystem metadata on books
  that have not changed since the last export.  (Does not depend on
  dates in the Calibre DB, exports all marked books and uses file-size
  comparisons.)

- Fixes up exported filenames to remove Goofy Characters That
  Shouldn't Be In Filenames.

- Uses an optional Genre (or other user-specified column) to organize
  books by sub-folders in the export directory.

## Dependencies

Tested with Calibre v1.26 to v1.27.  calibre-book-export itself is a
Bash script, and it relies on Rsync.  (My apologies to Windows folks,
but this is the world I live in.)

## Application Example

The author wants certain books in his Calibre library to automagically
appear on his Android tablet, by way of sychronizing a folder from his
personal webserver.  Calibre Book Export, run on the workstation where
Calibre is installed, searches the Calibre DB for books tagged/flagged
as "export" (he uses a custom column named "Fav" to keep track of
things to read next, and one of the values possible in this column is
"export"), and exports the marked books into a *temporary* directory.
CBE then uses Rsync to sync this temporary directory to the "real"
target directory.  That target directory is later Rsync'd by a cronjob
(not part of this package) to the remote webserver directory.

The key element here is that tempdir-to-target Rsync is set to *not*
update timestamps and to delete missing files in the target.  This
means it only changes files which have actually changed, and doesn't
change timestamps unnecessarily.  This is important because some sync
tools rely entirely on date to tell that a file has changed, and even
Rsync defaults to depending on the date.

In addition to the above, Calibre Book Export looks at the Genre
column (another custom column, with genre entries like
Tech.Programming.Bash) and makes sub-directories in the target
directory based on the first two elements of the first genre
(books/Tech.Programming), filing all the exported books appropriately.

## Limitations and Known Bugs

*Redundant Export*: At the moment, calibre-book-export exports every
book marked for export and *then* does the comparison with the target
directory.  This works well for small export sets, but may be
cumbersome to, say, export all the books from a large database on a
nightly basis.  It will work, and if it happens when you're asleep you
likely won't notice.  But it will go through the motions of writing
out every book even if no book has changed in weeks.

This is because the script doesn't care what is in the Calibre DB
except that a book is flagged for export, and the optional value to
file the book in a sub-directory.  calibre-book-export keeps no state
of its own, beyond the exported book directory.  Replacing a book
format within a book record does *not* update the "Date" meta-data
(which normally reflects when the book record was created).  So it's
not a reliable indicator of whether a book has changed or not, even if
I thought it worthwhile to track the state necessary to determine if a
book has already been exported or not.

If there is any demand for it, I'll consider adding it.  But to
reliably detect book changes probably means hunting down the book in
the Calibre library directory and checking the date on the actual
file.  Finding that book likely will depend on reading the Calibre
SQLite database directly to read the book path.  That's a lot of work
and debugging to avoid having the script just write everything out
every night.

Optionally, calibre-book-export could keep track of what books were
marked for export last time and only export books that were marked for
export since the last run.  This would work for new books, which has
been the bulk of my usage, but it wouldn't work for books that have
been updated.  (That happens when beta copies are revised, or books
get updated with errata, etc.  I do deal with this, and having updated
books reexported is a necessary feature for me.)
