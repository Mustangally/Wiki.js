---
title: Relocating the Home Directory
description: 
published: true
date: 2022-03-19T20:01:02.398Z
tags: linux
editor: markdown
dateCreated: 2022-03-19T20:01:00.096Z
---

<h2 id="bkmrk-why-keep-your-home-f" role="heading" aria-level="2">Why Keep Your home Folder Separate?</h2>
<p id="bkmrk-if-you%E2%80%99re-setting-up">If you’re setting up a new machine or adding a hard drive to an existing one, you may want to have your home directory on a different drive than the default location.</p>
<p id="bkmrk-an-increasingly-popu">An increasingly popular configuration for modern personal computers is to have a medium-sized <a href="https://www.howtogeek.com/howto/45359/htg-explains-whats-a-solid-state-drive-and-what-do-i-need-to-know/">Solid State Drive</a> (SSD) holding your operating system and a larger <a href="https://www.howtogeek.com/195262/hybrid-hard-drives-explained-why-you-might-want-one-instead-of-an-ssd/">Solid State Hybrid Drive</a> (SSHD) or traditional hard drive (HD) as your the main storage for data. Or you may have a single traditional hard drive in your system, and you’ve added a new HD for increased storage. Whatever your reasons, here is a simple and blow by blow run-through of moving your home directory.</p>
<p id="bkmrk-by-the-way%2C-if-you%E2%80%99r">By the way, if you’re installing a Linux system from scratch, you’ll probably see an option to create a separate home directory in your Linux distribution’s installer. Generally, you’ll just need to go into the partitioning options, create a separate partition, and mount it at “/home”. But, if you’ve already installed a Linux distribution, you can use these instructions to move your current home directory to a new location without losing anything or reinstalling your operating system.</p>
<h2 id="bkmrk-identify-the-drive" role="heading" aria-level="2">Identify the Drive</h2>
<p id="bkmrk-if-you%E2%80%99ve-just-fitte">If you’ve just fitted a drive to a Linux computer, or installed Linux to one of the drives in a new multi-drive computer, and rebooted, there’s little evidence that the new drive is even present.</p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-the%C2%A0fdisk%C2%A0command-wi">The <code>fdisk</code> command will <a href="http://man7.org/linux/man-pages/man8/fdisk.8.html">list the drives and their partitions</a> for us.</p>
<pre id="bkmrk-sudo-fdisk--l"><code class="language-">sudo fdisk -l</code></pre>
<p id="bkmrk-"><img class="alignnone size-full wp-image-442163" src="https://www.howtogeek.com/wp-content/uploads/2019/09/1-12.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-scroll-through-the-o">Scroll through the output until you have identified the new drive. The first drive is named <code>/dev/sda</code> , the second is <code>/dev/sdb</code> and so on, with the last letter increasing each time. So <code>/dev/sde</code> would be the fifth hard drive in the system.</p>
<p id="bkmrk-in-this-example%2C-the">in this example, the new drive is the second drive to be fitted to the system. So we need to look for an entry for <code>/dev/sdb</code>.</p>
<p id="bkmrk--0"><img class="alignnone size-full wp-image-442166" src="https://www.howtogeek.com/wp-content/uploads/2019/09/2-11.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-%2Fdev%2Fsdb%C2%A0is-highligh"><code>/dev/sdb</code> is highlighted above. You’ll notice that it doesn’t have a line describing a partition on it. It’s a brand new drive so it won’t have one yet. We need to create the partition. We can do so using <code>fdisk</code>. If your hard drive is not <code>/dev/sdb</code>, make sure you substitute <code>/dev/sdb</code> with the actual drive identifier for your new hard drive in the command.</p>
<pre id="bkmrk-sudo-fdisk-%2Fdev%2Fsdb"><code class="language-">sudo fdisk /dev/sdb</code></pre>
<p id="bkmrk--1"><img class="alignnone size-full wp-image-442173" src="https://www.howtogeek.com/wp-content/uploads/2019/09/3-13.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-when%C2%A0fdisk%C2%A0prompts-y">When <code>fdisk</code> prompts you for a command, press the letter <code>p</code>. This prints the partition table for the hard drive. We know it won’t have one, but we get some useful information about the drive. It gives us a good chance to make sure that the drive we’re going to create a partition for is the drive we intended to work with.</p>
<p id="bkmrk-it-tells-us-that-the">It tells us that the drive is a 1TB drive, which matches what we expect in this test machine, so we’ll proceed.</p>
<h2 id="bkmrk-create-a-partition" role="heading" aria-level="2">Create a Partition</h2>
<p id="bkmrk-press-the-letter%C2%A0n%C2%A0f">Press the letter <code>n</code> for a new partition, and then press <code>p</code> for a primary partition. When you are asked for the partition number, press the number <code>1</code>.</p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-0"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-we%E2%80%99re-going-to-creat">We’re going to create a single partition for the whole disk, so when prompted for the first sector we can press Enter to accept the default value. You will then be prompted for the last sector, and Enter will accept the default value.</p>
<p id="bkmrk--2"><img class="alignnone size-full wp-image-442176" src="https://www.howtogeek.com/wp-content/uploads/2019/09/4-7.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-although%C2%A0fdisk%C2%A0confi">Although <code>fdisk</code> confirms that it has created a 1TB Linux partition, which is partition number 1, nothing has changed on the hard drive yet. Until you give <code>fdisk</code> the command to write the changes to the drive, the drive is untouched. Once you are certain you’re happy with our choices, press the letter <code>w</code> to write the changes to the drive.</p>
<p id="bkmrk--3"><img class="alignnone size-full wp-image-442182" src="https://www.howtogeek.com/wp-content/uploads/2019/09/5-7.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-the-partition-has-be">The partition has been written to <code>/dev/sdb</code> . Let’s check what just happened. We’ll use <code>fdisk</code> once more on <code>/dev/sdb</code>.</p>
<pre id="bkmrk-sudo-fdisk-%2Fdev%2Fsdb-0"><code class="language-">sudo fdisk /dev/sdb</code></pre>
<p id="bkmrk--4"><img class="alignnone size-full wp-image-442183" src="https://www.howtogeek.com/wp-content/uploads/2019/09/7-7.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-press-the-letter%C2%A0p%C2%A0t">Press the letter <code>p</code> to print that partition table, and you’ll see that there is a partition listed for the drive now. Because it was the first partition on this drive, it is called <code>/dev/sdb1</code>. A second partition would be called <code>/dev/sdb2</code>, and so on.</p>
<p id="bkmrk-we-don%E2%80%99t-want-to-mak">We don’t want to make any changes to the partition, so press the letter <code>q</code> to quit.</p>
<h2 id="bkmrk-create-a-file-system" role="heading" aria-level="2">Create a File System on the Partition</h2>
<p id="bkmrk-we-need-to-create-a-">We need to create a filesystem on the partition. This is easily achieved with the <code>mkfs</code> command. Note that you must include the partition number <a href="http://man7.org/linux/man-pages/man8/mkfs.8.html">in the command</a>. Be careful to type <code>/dev/sdb1</code> (the partition) and not <code>/dev/sdb</code> (the drive).</p>
<pre id="bkmrk-sudo-mkfs--t-ext4-%2Fd"><code class="language-">sudo mkfs -t ext4 /dev/sdb1</code></pre>
<p id="bkmrk--5"><img class="alignnone size-full wp-image-442187" src="https://www.howtogeek.com/wp-content/uploads/2019/09/8-8.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-1"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-the-filesystem-will-">The filesystem will be created for you, and you’ll be returned to the command prompt.</p>
<p id="bkmrk--6"><img class="alignnone size-full wp-image-442188" src="https://www.howtogeek.com/wp-content/uploads/2019/09/9-6.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="317"></p>
<h2 id="bkmrk-mounting-the-new-dri" role="heading" aria-level="2">Mounting the New Drive</h2>
<p id="bkmrk-to-use-the-new-drive">To use the new drive, we must mount the partition on it to a mount point in the filesystem. Actually, to be perfectly accurate, we’re neither mounting the drive nor the partition, we’re mounting the <em>filesystem</em> on the partition, by grafting it onto your system’s <em>filesystem</em> tree.</p>
<p id="bkmrk-the%C2%A0%2Fmnt%C2%A0point-is-as">The <code>/mnt</code> point is as good a place as any. It is only a temporary mount point to allow us to copy data to the new drive. We’re going to use the <code>mount</code> command to <a href="https://www.howtogeek.com/414634/how-to-mount-and-unmount-storage-devices-from-the-linux-terminal/">mount the filesystem</a> on the first partition on <code>/dev/sdb</code>, at <code>/mnt</code> .</p>
<pre id="bkmrk-sudo-mount-%2Fdev%2Fsdb1"><code class="language-">sudo mount /dev/sdb1 /mnt</code></pre>
<p id="bkmrk--7"><img class="alignnone size-full wp-image-442190" src="https://www.howtogeek.com/wp-content/uploads/2019/09/10-6.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-if-all-goes-well%2C-yo">If all goes well, you’ll be returned to the command line with no error messages. Let’s see if we can change directory to our newly mounted filesystem.</p>
<pre id="bkmrk-cd-%2Fmnt"><code class="language-">cd /mnt</code></pre>
<p id="bkmrk--8"><img class="alignnone size-full wp-image-442191" src="https://www.howtogeek.com/wp-content/uploads/2019/09/11-6.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="77"></p>
<p id="bkmrk-yes%2C-we-can.-let%E2%80%99s-s">Yes, we can. let’s see what’s here.</p>
<pre id="bkmrk-ls--ahl"><code class="language-">ls -ahl</code></pre>
<p id="bkmrk--9"><img class="alignnone size-full wp-image-442192" src="https://www.howtogeek.com/wp-content/uploads/2019/09/12-7.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="152"></p>
<p id="bkmrk-we%E2%80%99re-in-our-new-fil">We’re in our new file system. The default “lost+found” directory is not required so we can remove it.</p>
<pre id="bkmrk-sudo-rm--rf-lost%2Bfou"><code class="language-">sudo rm -rf lost+found</code></pre>
<p id="bkmrk--10"><img class="alignnone size-full wp-image-442194" src="https://www.howtogeek.com/wp-content/uploads/2019/09/13-4.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<h2 id="bkmrk-copying-your-home-fo" role="heading" aria-level="2">Copying Your Home Folder</h2>
<p id="bkmrk-we-need-to-copy-ever">We need to copy everything from the old home directory to the newly mounted filesystem. Using the <code>r</code> (recursive) and <code>p</code> (preserve) options will ensure all <a href="http://man7.org/linux/man-pages/man1/cp.1.html">subdirectories are copied</a> and that file ownerships, permissions, and other attributes are retained.</p>
<pre id="bkmrk-sudo-cp--rp-%2Fhome%2F%2A-"><code class="language-">sudo cp -rp /home/* /mnt</code></pre>
<p id="bkmrk--11"><img class="alignnone size-full wp-image-442220" src="https://www.howtogeek.com/wp-content/uploads/2019/09/14-4.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-2"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-when-the-copy-has-co">When the copy has completed, use <code>ls</code> to have a look around and verify that your data is where you expect it to be in the new filesystem. In other words, if <code>/mnt</code> was your home directory, is everything present and correct?</p>
<pre id="bkmrk-ls"><code class="language-">ls</code></pre>
<pre id="bkmrk-ls-dave"><code class="language-">ls dave</code></pre>
<p id="bkmrk--12"><img class="alignnone size-full wp-image-442221" src="https://www.howtogeek.com/wp-content/uploads/2019/09/15-5.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="162"></p>
<p id="bkmrk-you%E2%80%99ll-probably-want">You’ll probably want to be a bit more thorough than we were on the test machine this article was researched on. As a safety net, we’re going to rename and keep your old <code>/home</code> directory until you’re satisfied that it is safe to delete it.</p>
<pre id="bkmrk-sudo-mv-%2Fhome-%2Fhome."><code class="language-">sudo mv /home /home.orig</code></pre>
<p id="bkmrk--13"><img class="alignnone size-full wp-image-442224" src="https://www.howtogeek.com/wp-content/uploads/2019/09/26-2.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-and-we%E2%80%99ll-create-a-n">And we’ll create a new, empty home directory.</p>
<pre id="bkmrk-sudo-mkdir-%2Fhome"><code class="language-">sudo mkdir /home</code></pre>
<p id="bkmrk--14"><img class="alignnone size-full wp-image-442225" src="https://www.howtogeek.com/wp-content/uploads/2019/09/27-2.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-we%E2%80%99ll-use-that-new-e">We’ll use that new empty home directory as the mount point for our filesystem on the new hard drive. We need to unmount it from <code>/mnt</code> and remount it on <code>/home</code>. Note that the command <code>umount</code> doesn’t have an “n” after the “u.”</p>
<p id="bkmrk-but-first%2C-we%E2%80%99ll-cha">But first, we’ll change into the root directory (with <code>cd /</code> ) to make sure we’re not in a directory that is going to be included in the mount or unmount locations.</p>
<pre id="bkmrk-cd-%2F"><code class="language-">cd /</code></pre>
<pre id="bkmrk-sudo-umount-%2Fdev%2Fsdb"><code class="language-">sudo umount /dev/sdb1</code></pre>
<pre id="bkmrk-sudo-mount-%2Fdev%2Fsdb1-0"><code class="language-">sudo mount /dev/sdb1 /home/</code></pre>
<p id="bkmrk--15"><img class="alignnone size-full wp-image-442226" src="https://www.howtogeek.com/wp-content/uploads/2019/09/16-4.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="97"></p>
<p id="bkmrk-related%3A%C2%A0the-linux-d"><strong>RELATED:</strong> <a href="https://www.howtogeek.com/117435/htg-explains-the-linux-directory-structure-explained/"><strong><em>The Linux Directory Structure, Explained</em></strong></a></p>
<h2 id="bkmrk-testing-your-new-hom" role="heading" aria-level="2">Testing Your New home Directory</h2>
<p id="bkmrk-let%E2%80%99s-see-what-the-a">Let’s see what the attributes of the <code>/dev/sdb1</code> partition are now:</p>
<pre id="bkmrk-df-%2Fdev%2Fsdb1"><code class="language-">df /dev/sdb1</code></pre>
<p id="bkmrk--16"><img class="alignnone size-full wp-image-442228" src="https://www.howtogeek.com/wp-content/uploads/2019/09/17-5.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="122"></p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-3"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-we%E2%80%99re-shown-the-name">We’re shown the name of the filesystem, the size of the partition and the used and available space on it, and importantly, where it is mounted. It is now our <code>/home</code> directory.  That means we should be able to reference it exactly as we could the old <code>/home</code> directory.</p>
<p id="bkmrk-if-we-move-to-some-a">If we move to some arbitrary point in the filesystem, we ought to be able to change back to <code>/home</code> using the <code>~</code> tilde shortcut.</p>
<pre id="bkmrk-cd-%2F-0"><code class="language-">cd /</code></pre>
<pre id="bkmrk-cd-%7E"><code class="language-">cd ~</code></pre>
<pre id="bkmrk-pwd"><code class="language-">pwd</code></pre>
<pre id="bkmrk-ls-0"><code class="language-">ls</code></pre>
<p id="bkmrk--17"><img class="alignnone size-full wp-image-442229" src="https://www.howtogeek.com/wp-content/uploads/2019/09/18-5.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="197"></p>
<pre id="bkmrk-cd-%2Fhome"><code class="language-">cd /home</code></pre>
<pre id="bkmrk-ls-1"><code class="language-">ls</code></pre>
<pre id="bkmrk-cd-dave"><code class="language-">cd dave</code></pre>
<pre id="bkmrk-ls-2"><code class="language-">ls</code></pre>
<pre id="bkmrk-ls--a"><code class="language-">ls -a</code></pre>
<p id="bkmrk--18"><img class="alignnone size-full wp-image-442230" src="https://www.howtogeek.com/wp-content/uploads/2019/09/19-5.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="382"></p>
<p id="bkmrk-we-can-move-through-">We can move through the filesystem back and forth to <code>/home</code> using explicit commands and using the <code>~</code> shortcut. The folders, files, and dotfiles we’d expect are all present. It’s all looking good.</p>
<p id="bkmrk-if-anything-was-miss">If anything was missing, we could copy it out of the <code>/home.orig</code> directory, which we still have access to in the root of the filesystem. But it all looks fine.</p>
<p id="bkmrk-now-we-need-to-have%C2%A0">Now we need to have <code>/dev/sdb1</code> mounted automatically every time your computer is started.</p>
<h2 id="bkmrk-editing-fstab" role="heading" aria-level="2">Editing fstab</h2>
<p id="bkmrk-the-%E2%80%9Cfstab%E2%80%9D-file-con">The “fstab” file contains descriptions of the filesystems that are going to be mounted when the system boots. Before we make any changes to it, we’ll make a backup copy of it that we can return to in the event of problems.</p>
<pre id="bkmrk-sudo-cp-%2Fetc%2Ffstab-%2F"><code class="language-">sudo cp /etc/fstab /etc/fstab.orig</code></pre>
<p id="bkmrk--19"><img class="alignnone size-full wp-image-442233" src="https://www.howtogeek.com/wp-content/uploads/2019/09/20-3.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-4"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-now-we-can-edit-the-">Now we can edit the fstab file. Use your favorite editor, we’re using <code>gedit</code>. Any text editor willdo.</p>
<pre id="bkmrk-sudo-gedit-%2Fetc%2Ffsta"><code class="language-">sudo gedit /etc/fstab</code></pre>
<p id="bkmrk--20"><img class="alignnone size-full wp-image-442234" src="https://www.howtogeek.com/wp-content/uploads/2019/09/21-4.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-you-must-add-a-line-">You must add a line at the bottom of the file to mount our new <code>/home</code> directory. If your drive and partition identifiers are different than the ones used in this example, substitute those for the <code>/dev/sdb1</code> shown here.</p>
<ul id="bkmrk-type-the-name-of-the">
<li>Type the name of the partition at the start of the line, and then press Tab.</li>
<li>Type the mount point, <code>/home</code>,  and press Tab.</li>
<li>Type the filesystem description <code>ext4</code>, and press Tab.</li>
<li>Type <code>defaults</code> for the mount options, and press Tab.</li>
<li>Type the digit <code>0</code> for the filesystem dump option, and press Tab.</li>
<li>Type the digit <code>0</code> for the filesystem check option.</li>
</ul>
<p id="bkmrk--21"><img class="alignnone size-full wp-image-442235" src="https://www.howtogeek.com/wp-content/uploads/2019/09/22-3.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="Using gedit to edit the fstab file" width="646" height="286"></p>
<p id="bkmrk-save-the-fstab-file.">Save the fstab file.</p>
<h2 id="bkmrk-reboot-your-system" role="heading" aria-level="2">Reboot Your System</h2>
<p id="bkmrk-we-need-to-reboot-to">We need to reboot to verify that everything has gone according to plan and that you have a seamless connection to your new <code>/home</code> directory.</p>
<p id="bkmrk-if-it-doesn%E2%80%99t%2C-you%E2%80%99v">If it doesn’t, you’ve still got the safety net of your original <code>/home</code> directory and fstab file that could be restored if required. Because of the precautions we’ve taken—copying the <code>/home</code> directory and fstab files—you could easily return your system to the state it was in before you started.</p>
<pre id="bkmrk-sudo-reboot-now"><code class="language-">sudo reboot now</code></pre>
<p id="bkmrk--22"><img class="alignnone size-full wp-image-442236" src="https://www.howtogeek.com/wp-content/uploads/2019/09/23-2.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="57"></p>
<p id="bkmrk-related%3A%C2%A0how-to-rebo"><strong>RELATED:</strong> <a href="https://www.howtogeek.com/411925/how-to-reboot-or-shut-down-linux-using-the-command-line/"><strong><em>How to Reboot or Shut Down Linux Using the Command Line</em></strong></a></p>
<h2 id="bkmrk-final-checks" role="heading" aria-level="2">Final Checks</h2>
<p id="bkmrk-when-your-system-res">When your system restarts let’s just check that your <code>/home</code> directory is really on your new hard drive, and your system hasn’t somehow (miraculously) reverted to using the old <code>/home</code> directory.</p>
<pre id="bkmrk-df-%2Fdev%2Fsdb1-0"><code class="language-">df /dev/sdb1</code></pre>
<p id="bkmrk--23"><img class="alignnone size-full wp-image-442238" src="https://www.howtogeek.com/wp-content/uploads/2019/09/24-2.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="122"></p>
<p id="bkmrk-great%2C-it%E2%80%99s-mounted-">Great, it’s mounted on <code>/home</code>. Mission accomplished.</p>
<p id="bkmrk-once-you%E2%80%99re-perfectl">Once you’re perfectly sure that you no longer need the safety copy of your old <code>/home</code> directory, you can delete it:</p>
<pre id="bkmrk-cd-%2F-1"><code class="language-">cd /</code></pre>
<pre id="bkmrk-sudo-rm--rf-home.ori"><code class="language-">sudo rm -rf home.orig/</code></pre>
<p id="bkmrk--24"><img class="alignnone size-full wp-image-442240" src="https://www.howtogeek.com/wp-content/uploads/2019/09/25-2.png?trim=1,1&amp;bg-color=000&amp;pad=1,1" alt="" width="646" height="77"></p>
<div data-nosnippet="data-nosnippet" id="bkmrk-advertisement-5"><span class="future_inline_clone_target">ADVERTISEMENT<br></span></div>
<p id="bkmrk-and-of-course%2C-if-yo">And of course, if you do realize something <em>didn’t</em> copy over from the old <code>/home</code> to your new <code>/home</code>, you’ll be able to retrieve it from the backup you made before we started.</p>
<h2 id="bkmrk-home-sweet-home" role="heading" aria-level="2">Home Sweet Home</h2>
<p id="bkmrk-now-that-you%E2%80%99ve-sepa">Now that you’ve separated your <code>/home</code> directory from the rest of the operating system’s partition, you can re-install your operating system, and your data will be untouched. All you have to do is edit the fstab file to mount your second drive on <code>/home</code>.</p>
<p id="bkmrk-and-because-all-of-y">And because all of your dotfiles are in your <code>/home</code> directory, when you fire up your various applications, they’ll find all of your settings, preferences, and data.</p>
<p id="bkmrk-it-takes-the-pain-ou">It takes the pain out of reinstalls and takes the risk out of upgrades.</p>