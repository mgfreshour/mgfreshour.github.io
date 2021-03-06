---
layout: post
title: Levenshtein Distance in Javascript
date: '2011-05-28T11:16:00.000-07:00'
author: Morgan Freshour
tags:
- javascript
- algorithm
modified_time: '2011-05-28T11:16:42.946-07:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-496456084922246857
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/05/levenshtein-distance-in-javascript.html
---

Okay, now this is an odd code snippet.  I really can't remember why I needed a javascript implementation of the [Levenshtein Distance](http://en.wikipedia.org/wiki/Levenshtein_distance) algorithm.  I mean, most back end languages have an implementation in their libraries that'd be far more efficient.  Oh well, since I saved it, I might as well share it.

{% highlight javascript %}
/**
 * Calculates the levenshtein distance between two strings
 * @param string string1
 * @param string string2
 */
function levenshtein(string1, string2)
{
    //declare int d[0..m, 0..n]
    var width = string1.length+1;
    var height = string2.length+1;
    var dtable = new Array(width * height);
    var i,j;

    // IE doesn't like element access to string
    string1 = string1.split("");
    string2 = string2.split("");

    //for i from 0 to m
    for (i=0; i < width; i++) {
        //d[i, 0] := i // deletion
        dtable[i] = i;
    }
    //for j from 0 to n
    for (j=0; j<height; j++) {
        //d[0, j] := j // insertion
        dtable[j*width] = j;
    }

    //for j from 1 to n
    for (j=1; j<height; j++) {
        //for i from 1 to m
        for (i=1; i<width; i++) {
            //if s[i] = t[j] then
            if (string1[i-1] == string2[j-1]) {
                //d[i, j] := d[i-1, j-1]
                dtable[i + j*width] = dtable[(i-1) + (j-1)*width];
            }
            //else
            else {
                //d[i, j] := minimum ( d[i-1, j] + 1,  // deletion
                //                     d[i, j-1] + 1,  // insertion
                //                     d[i-1, j-1] + 1 // substitution )
                dtable[i + j*width] = Math.min(
                    dtable[(i-1) +  j   *width]+1, // deletion
                    dtable[(i)   + (j-1)*width]+1, // insertion
                    dtable[(i-1) + (j-1)*width]+1  // substitution
                );
            }
        }
    }

    //console.log(print2dArray(dtable, width, height));

    //return d[m,n]
    return dtable[width*height - 1];
}

/**
 * Makes a pretty print of a 2d array
 */
function print2dArray(table, width, height) {
    var i,j,out = "";
    out += "\n";
    for (i=0; i<width; i++) {
        for (j=0; j<height; j++) {
            out += "[" + table[i + j*width] + "]";
        }
        out += "\n";
    }

    return out;
}

{% endhighlight %}
