---
layout: post
title: Case Insensitive Diff Algorithm in PHP
date: '2011-05-28T11:19:00.000-07:00'
author: Morgan Freshour
tags:
- algorithm
- php
modified_time: '2011-05-28T11:19:54.551-07:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-4465721473758812964
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/05/case-insensitive-diff-algorithm-in-php.html
---

So, one day the business analyst wanted to compare what people where typing in as their address and the corrected/validated results we use.  My first thought was "Oh that'd be easy, we just use a <a href="http://en.wikipedia.org/wiki/Diff">diff</a> function on a report!"  What I didn't think about was that most people type in the lowercase, but the corrected address are in all caps (as per <a href="http://pe.usps.com/text/pub28/welcome.htm">Post Office specs</a>).


Okay, so we just need a case insensitive diff algorithm, but wait, there's more.  "Marshal Hollow Rd" and "Maral Hollow Rd" only show up as a tiny difference, but obviously they're completely different addresses.  We needed to highlight the differences in the words more.  So, now we need a case insensitive diff that judges differences based on words rather than characters.


What follows was my solution to this need.  I wrote this a number of years ago, and I no longer remember much about it.  I do know that I did *not* invent the diff algorithm.  There was a comment with this <a href="http://www.somethinkodd.com/oddthinking/2006/01/16/comparing-strings-an-analysis-of-diff-algorithms/">link</a> in it. Likely I just wrote what is said on that page and modified it to handle our special needs.  If I read the website in the comment and studied the code, I could probably regain my understanding of this algorithm, but I don't think I really need to understand it anymore as in the intervening years I've not needed a similar algorithm since.


Here's the code!

{% highlight php %}
<?php
function htmlWordDiff($old, $new, $diff_prefix='<span style="background:#80ff80;">', $diff_suffix='<span>') {
    $cap_old = explode(' ', strtoupper($old));
    $cap_new = explode(' ', strtoupper($new));
    $orig_new = explode(' ', $new);
    $orig_old = explode(' ', $old);
    $translate_new = array();
    $translate_old = array();
    for ($n = 0; $n < sizeof($orig_new); $n++) {
        $translate_new[$cap_new[$n]] = $orig_new[$n];
    }
    for ($n = 0; $n < sizeof($orig_old); $n++) {
        $translate_old[$cap_old[$n]] = $orig_old[$n];
    }
    
    $differences = $this->wordDiff($cap_old, $cap_new);
    
    foreach ($differences as $diff) {
        if (is_array($diff)) {
            if (! empty($diff['d'])) {
                //$ret .= "<del>".implode(' ', $diff['d'])."</del> ";
            }
            if (! empty($diff['i'])) {
                $new_diff = array();
                $ret .= $diff_prefix;
                foreach ($diff['i'] as $word) {
                    $new_diff[] = $translate_new[$word];
                }
                $ret .= implode(' ', $new_diff);
                $ret .= $diff_suffix;
            }
        } else
            $ret .= $translate_new[$diff].' ';
    }
    return $ret;
}

function wordDiff($old, $new) {
    foreach ($old as $oindex=>$ovalue) {
        $nkeys = array_keys($new, $ovalue);
        foreach ($nkeys as $nindex) {
            $matrix[$oindex][$nindex] = isset($matrix[$oindex - 1][$nindex - 1]) ? $matrix[$oindex - 1][$nindex - 1] + 1 : 1;
            if ($matrix[$oindex][$nindex] > $maxlen) {
                $maxlen = $matrix[$oindex][$nindex];
                $omax = $oindex + 1 - $maxlen;
                $nmax = $nindex + 1 - $maxlen;
            }
        }
    }
    
    if ($maxlen == 0) {
        return array(array('d'=>$old, 'i'=>$new));
    }
    
    return array_merge($this->wordDiff(array_slice($old, 0, $omax), array_slice($new, 0, $nmax)), array_slice($new, $nmax, $maxlen), 
                       $this->wordDiff(array_slice($old, $omax + $maxlen), array_slice($new, $nmax + $maxlen)));
}
{% endhighlight %}