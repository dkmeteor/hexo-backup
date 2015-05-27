title: How to soleve eclipse refresh external folder
id: 140
categories:
  - android
date: 2014-11-21 08:33:53
tags:
---

### How to soleve eclipse refresh external folder

According to the stackoverflow:

> If you have references to external sources put them in a zip file:
> YourProject-&gt;rightClick-&gt;Properties-&gt;Java Build Path-&gt;libraries-&gt;..., and then most notably android.jar, but other libs can be the culprit too. Expand it and and select Source attachment, and then (if it doesn't say 'None') press the 'Edit...'-button. If that points to a directory waht you should do is compress that **source-directory** into a zip file and make the source attachment point to that file.Apparently eclipse/adt feels the need to refresh sources on the file-system. When they're in a zip-file it seems confident that they have not changed....

Just unattached source code is OK.