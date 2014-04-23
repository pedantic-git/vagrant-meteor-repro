# Reproduction of Meteor Vagrant issue

This is a reproduction of Meteor issue [1337](https://github.com/meteor/meteor/issues/1337).

## How to reproduce

1. Clone this repo.
2. Install [Virtualbox](https://www.virtualbox.org/).
3. Install [Vagrant](http://www.vagrantup.com/).
4. Change to this directory
5. `vagrant up`
6. `vagrant ssh`
7. `cd /vagrant/leaderboard`
8. `meteor`
9. Check the leaderboard is accessible at http://localhost:3000/ on
   your host machine
10. Change the leaderboard.js file while Meteor is running
11. See Meteor fall over

## What just happened

Vagrant created a virtual machine using the official Trusty Tahr image
from Ubuntu and then ran the shell script at the bottom of
`Vagrantfile`.

The git repo is mounted on the guest box as `/vagrant` using the
vboxsf (Virtual Box Shared Folders) filesystem, and port 3000 is
forwarded to the host machine.

The script had to install MongoDB because Meteor refuses to run Mongo
from the `/vagrant` filesystem (it doesn't support locking).

When we tried to change a file while Meteor was running, issue 1337
was exposed - specifically it tried to remove a directory before that
directory was empty.

## Workaround

Replacing the call to files.renameDirAlmostAtomically in builder.js
with a symlink works around the issue but the files in
`.meteor/local/build` are not properly cleaned up, so a more permanent
fix is advisable.

The workaround patch is included below for reference:

```diff
diff -uNPr /home/vagrant/tools-orig/latest/tools/builder.js /home/vagrant/.meteor/tools/latest/tools/builder.js
--- /home/vagrant/tools-orig/latest/tools/builder.js	2014-03-06 17:23:26.304660956 +0000
+++ /home/vagrant/.meteor/tools/latest/tools/builder.js	2014-03-06 17:11:56.568650985 +0000
@@ -435,7 +435,7 @@
     // outputPath be a symlink pointing to it. This doesn't work for the NPM use
     // case of renameDirAlmostAtomically since that one is constructing files to
     // be checked in to version control, but here we could get away with it.
-    files.renameDirAlmostAtomically(self.buildPath, self.outputPath);
+    files.symlink(self.buildPath, self.outputPath);
   },
 
   // Delete the partially-completed bundle. Do not disturb outputPath.
diff -uNPr /home/vagrant/tools-orig/latest/tools/files.js /home/vagrant/.meteor/tools/latest/tools/files.js
--- /home/vagrant/tools-orig/latest/tools/files.js	2014-02-26 21:52:58.000000000 +0000
+++ /home/vagrant/.meteor/tools/latest/tools/files.js	2014-03-06 17:22:06.912923040 +0000
@@ -428,6 +428,11 @@
   future.wait();
 };
 
+files.symlink = function (fromDir, toDir) {
+  try { fs.unlinkSync(toDir); } catch(c) { }
+  fs.symlinkSync(fromDir, toDir);
+};
+
 // Use this if you'd like to replace a directory with another
 // directory as close to atomically as possible. It's better than
 // recursively deleting the target directory first and then
```
