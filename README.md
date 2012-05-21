fmscripts
=========

Using FileMerge as a diff command for Subversion

---

These scripts were created by Bruno De Fraine. The official website (and official version) can be found at: http://www.defraine.net/~brunod/fmdiff/

<hr>

<h2>Introduction</h2>
<p>
Subversion principally uses the <a href="http://svnbook.red-bean.com/en/1.5/svn.basic.vsn-models.html#svn.basic.vsn-models.copy-merge">copy-modify-merge</a> versioning model to allow concurrent collaboration on the same set of files.
For this to work well, it is crucial to have good tools to view and merge the differences between files.
</p>
<p>
The built-in <code>diff</code> and <code>diff3</code> tools (from <a href="http://www.gnu.org/software/diffutils/diffutils.html">diffutils</a>) go a long way to provide this functionality for text files on the command line.
They do have some limitations though, including:
</p>
<ul>
<li>Worthless to find interline changes in long lines</li>
<li>Output of diff can be hard to read</li>
<li>No interactive merge support</li>
</ul>
<p>
Apple's <a href="http://developer.apple.com/technology/">Developer Tools</a> for Mac OS X include FileMerge, a graphical tool to compare and merge files.
FileMerge can be much handier to use; unfortunately, it doesn't integrate with Subversion straightforwardly.
It can be opened from the command line with the <code>opendiff</code> command, but its interface differs from that of <code>diff</code> and <code>diff3</code>.
It returns immediately (i.e. it forks and does not block) and it expects different arguments.
Some wrapper scripts are thus required to call FileMerge from Subversion.
</p>
<p><img src="filemerge.gif" width="400" title="FileMerge in action" alt="FileMerge in action" /></p>
<h2>Wrapper Scripts for FileMerge</h2>
<p>
Bash scripts for this task can be checked-out from <a href="http://soft.vub.ac.be/svn-gen/bdefrain/fmscripts/">this repository</a> or downloaded as an <a href="fmscripts-20120521.tar.gz">archive</a>.
</p>
<p>
Four scripts are provided: <code>fmdiff</code>, <code>fmdiff3</code>, <code>fmresolve</code> and <code>fmmerge</code>.
They are described below.
The location of FileMerge is determined through a symbolic link in the PATH environment. The included Makefile can create this link by checking a number of standard locations:
<pre>
$ make
</pre>
Afterwards, you can do the following to install all of the scripts in <code>/usr/local/bin</code>:
</p>
<pre>
$ sudo make install
</pre>
<h3>fmdiff</h3>
<p>
<code>fmdiff</code> has an interface similar to <code>diff</code> and can be used with Subversion's <code>--diff-cmd</code> option.
The canonical case is:
</p>
<pre>
$ svn diff --diff-cmd fmdiff &lt;other diff options, files or URLs&gt;
</pre>
<p>
Subversion will start FileMerge for each file with differences.
It will wait for you to quit FileMerge (⌘-Q) to clean up and show the next file (if any).
The script cannot show the labels from Subversion in FileMerge.
As a resort, it will print these on the command line:
</p>
<pre>
Starting FileMerge...
Left: lock.c (revision 14670)
Right: lock.c (revision 14675)
</pre>
<p>
Additionally, if <code>fmdiff</code> finds that you have <code>growlnotify</code> installed, it will use this to show a similar message as a <a href="http://growl.info/">Growl</a> notification.
</p>
<p>
<strong>Advanced tip:</strong>
The merge pane at the bottom of the FileMerge window is normally not used for <code>fmdiff</code>.
However, when <code>fmdiff</code> detects that the right-hand file is part of a working copy, it will instruct FileMerge to use this right-hand file as the merge file as well.
This means that you can compose a new version of this file in the merge pane by accepting or rejecting the changes that have occurred.
When you instruct FileMerge to save (⌘-S), the right-hand file will be replaced with the new version from the merge pane.
</p>
<h3>fmdiff3</h3>
<p>
Unsurprisingly, <code>fmdiff3</code> is the <code>diff3</code>-equivalent of <code>fmdiff</code>.
It can be used with Subversion's <code>--diff3-cmd</code> option.
</p>
<p>
<code>diff3</code> will merge the changes from two different versions that have a common ancestor.
It is used by Subversion on an <code>update</code> (to merge changes from the repository with local changes) or a <code>merge</code> operation.
Suppose that you have made local changes in a working copy and that meanwhile the repository was updated.
To merge your local changes with those of the repository in FileMerge, do:
</p>
<pre>
$ svn update --diff3-cmd fmdiff3
</pre>
<p>
<strong>Note:</strong> By using the ancestor information, <code>diff3</code> can merge the changes automatically, provided that there are no conflicting changes.
The standard <code>diff3</code> merge is therefore non-interactive.
If conflicts do occur, they are marked in brackets, and Subversion expects you to <a href="http://svnbook.red-bean.com/en/1.5/svn.tour.cycle.html#svn.tour.cycle.resolve">resolve the conflicts</a> later on, after the (partial) merge.
<code>fmdiff3</code>, in contrast, does the merge interactively and completely.
First, It will fire up FileMerge with the relevant ancestor information, and FileMerge will merge the non-conflicting changes by itself on startup.
<code>fmdiff3</code> then expects that you complete the merge by resolving the conflicts through FileMerge's facilities.
When you save the merged version (⌘-S) and quit FileMerge (⌘-Q), the result is given back to Subversion as a completely merged version.
So the conflict state is not needed anymore.
</p>
<p><img src="filemerge_conflict.gif" width="400" title="FileMerge interactive merge with conflict" alt="FileMerge interactive merge with conflict"/></p>
<h3>fmresolve</h3>
<p>
Whereas <code>fmdiff3</code> can be useful when you anticipate conflicts during an <code>update</code> or a <code>merge</code>, it's sometimes unpractical that you have to decide to use FileMerge <em>a priori</em>.
When you did a standard <code>update</code> or <code>merge</code> using the built-in <code>diff3</code>, and you are faced with one or more files in conflicting state, <code>fmresolve</code> allows to use FileMerge <em>a posteriori</em> to resolve the conflicts.
</p>
<pre>
$ fmresolve &lt;conflictfile&gt;
</pre>
<p>
This will start FileMerge with the appropriate versions and ancestor to reinitiate the merge.
Non-conflicting changes are resolved automatically at startup; you can then resolve the remaining changes.
After you save (⌘-S) and quit FileMerge (⌘-Q), you can signal to Subversion that the conflicts were resolved:
</p>
<pre>
$ svn resolved &lt;conflictfile&gt;
</pre>
<h3>fmmerge (Subversion 1.5)</h3>
<p>
Starting from version 1.5, Subversion supports <a href="http://svnbook.red-bean.com/en/1.5/svn.tour.cycle.html#svn.tour.cycle.resolve">interactive conflict resolution</a>.
That means that the above description of an <code>update</code> scenario is no longer entirely accurate: when Subversion detects conflicting changes during an update, it will provide an interactive menu with several options.
If you select the <q>postpone</q> option (<code>p</code>), the old behavior is applied, and you may still use <code>fmresolve</code> to resolve the conflict later on.
</p>
<p>
However, you can also choose to resolve the conflict immediately, either by accepting one version entirely (and discarding the changes in the other version), or by launching an external tool with the <q>launch</q> option (<code>l</code>).
<code>fmmerge</code> is a wrapper to use FileMerge as such an external merge tool.
It can only be activated through the Subversion configuration file (see below), where it must be configured as the variable <code>merge-tool-cmd</code>.
</p>
<h2>Using FileMerge Permanently</h2>
<p>
You can configure the <code>fmdiff</code> scripts to be used permanently, by setting the options <code>diff-cmd</code> and <code>diff3-cmd</code> in the "editors" section of the runtime config file (i.e. <code>~/.subversion/config</code>).
After this change, you no longer have to include the <code>--diff-cmd</code> and <code>--diff3-cmd</code> options when invoking Subversion.
</p>
<p>
More info: <a href="http://svnbook.red-bean.com/en/1.5/svn.advanced.confarea.html#svn.advanced.confarea.opts.config">Subversion Book - Runtime Config Area</a>.
</p>
<h2>License</h2>
<p>
These scripts are released in the public domain. Feel free to use them for any purpose. All material is provided without warranty of any kind.
</p>
<h2>Feedback?</h2>
<p>
Send comments and bugs to <a href="../">Bruno De Fraine</a>.
</p>
<p>Last Modified: <script type="text/javascript">document.write(new Date(document.lastModified));</script></p>
