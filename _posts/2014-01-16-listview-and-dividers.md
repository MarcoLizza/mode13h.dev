---
title: ListView and Dividers
author: marco.lizza
layout: post
permalink: /listview-and-dividers/
categories:
  - android
  - development
  - ui-ux
---
I was struggling with the `ListView` component, trying to figure out why on Earth in one activity the items were correctly separated and in another the last item featured a (single line) divider.

*(as a sidenote, I wasn&#8217;t using listiview headers and/or footers that would cause the behaviour to happen)*

In the beginning I thought it would depend on the fact I was using a standard `ArrayAdapter` in the first listview and a custom one (derived from `BaseAdapter`) in the second.

Without any clue, I left the problem unravelled, and solved it by adding the `android:footerDividersEnabled="false"` option to the layout XML (or a `_listView.setFooterDividersEnabled(false);` statement).

Yesterday, while playing around with the debug flags of my android device I enabled the &#8220;Show layout bounds&#8221; developer option&#8230; and all of a sudden all was clear. The second listview bound were stretching to fit the parent height, too, and it was due to the following option

<pre class="EnlighterJSRAW" data-enlighter-language="xml" data-enlighter-highlight="8">&lt;ListView
 android:id="@+id/listView"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:layout_alignParentLeft="true"
 android:layout_alignParentRight="true"
 android:layout_alignParentTop="true"
 android:layout_alignParentBottom="true" /&gt;</pre>

Apparently, stretching the listview layout to fill the parent height **while** wrapping just it&#8217;s content caused the problem.

Of course the other listview wasn&#8217;t using this attribute&#8230; I should have checked earlier. 😛

*Edit 02/06/2014*

The above method seems to be only once consistent across every Android OS release (while the `footerDividersEnabled` is not).