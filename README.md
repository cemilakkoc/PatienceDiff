# PatienceDiff
A consice javascript implementation of the Patience Diff algorithm.

# Interface
### PatienceDiff(aLines, bLines)

<b>result = patienceDiff(aLines, bLines)</b>

where:<br>
* aLines[] = array of strings representing the original lines of text.
* bLines[] = array of strings representing the new lines of text.
    
* result[] = array of objects, with properties of:
  * line = line of text from either aLines or bLines.
  * aIndex = index of original line in aLines, or -1 if line is added from bLines.
  * bIndex = index of new line in bLines, or -1 if line is deleted from aLines.

[Sample use of javascript Patience Diff](PatienceDiff.png)

# Explanation of the Patience Diff Algorithm
The algorithm is best explained by describing the supporting algorithms.

### findUnique(arr, lo, hi) returns Map
The Patience Diff algorithm requires unique entries between the arrays being compared when applying the Longest Common Subsequence algorithm, and the findUnique function uses javascript's map method to calculate the count of the lines of equal value.  Ie, there's no sense in wasting cycles in finding the unique lines between array A and B until finding the unique lines in A and B individually.  After running through all the lines in the array, the Map will contain the line as the key, and an object with properties of the count of matching lines, and the index of the last matching line.  Then, the Map is walked to remove all entries except for those with count === 1.  Note that since this Patience Diff algorithm is recursive, that findUnique takes the lo and hi range of the array as input parameters.  This also eliminates the performance hit of creating slices of the A and B arrays. 

In terms of complexity, presumably the javascript Map function is based on a hashing algorithm, and therefore O(1) in execution, and therefore the findUnique function is O(N).  Or more accurately, O(Na) and O(Nb) for each array, but assuming the arrays ultimately being compared are generally equal in length, the complexity is O(N).  Additionally, walking the Map to eliminate all but count === 1 is also linear in time, therefore not affecting the complexity.

### uniqueCommon(aArray, aLo, aHi, bArray, bLo, bHi) returns Map
The uniqueCommon function does what it's named, and finds the unique common lines between arrays A and B.  This is accomplished first by calling findUnique individually for arrays A and B, at which point two Maps are in hand of the unique lines in A and unique lines in B.  Then array A is walked, looking for the corresponding line in array B.  If found, the A Map entry is supplemented with the corresponding matching B Map entry, otherwise the A Map entry is deleted as there is no matching unique line in B.  Again, this function takes lo and hi ranges, as the overall function is designed to be recursive.

In terms of complexity, once the A and B Maps are in hand, the uniqueCommon function simply walks the A Map seeking presumably via hash the B Map value, and therefore is also O(N) in a worse case situation in which arrays A and B completely match with unique lines.

### longestCommonSubsequence(abMap) returns Array
The longestCommonSubsequence algorithm is best described by the "Alfedenzo" artilce at https://alfedenzo.livejournal.com/170301.html.  Suffice to say, it walks the result of uniqueCommon to build a jagged array, and then walks this jagged array backwards to find the longest common subsequence.  Since the jagged array is walked backwards, it is reversed and then returned.

In terms of complexity, again this is O(N) worse case, as the three (3) steps are all single pass linear in nature: 1) building the jagged array, 2) walking the jagged array backwards, and 3) reversing the result before returning.

### addSubMatch(aLo, aHi, bLo, bHi) adds values to the "result" array
Here's where it gets fun.  The addSubMatch function is being told that A[Lo-aHi] and B[bLo-bHi] inclusive, represent a range between a pair of matching unique values discovered by the longestCommonSubsequence (LCS).  That being said, there are edge cases at the beginning and end of the LCS, in which there might be lines in either array A or B before the first unique common line is found, and the same goes for lines occurring after the last unique common line found.  And generally speaking, when the range between an LCS pair is passed, the lo entry is included but not hi entry, as the hi entry becomes the lo on the next iteration of calling addSubMatch.  Again, indexes to the A and B arrays are passed to addSubMatch to avoid the performance hit of creating slices.

By example, let's say for text line arrays A[0..22] and B[0..28], the LCS found the following three (3) unique matches:
- A[5]  === B[7]
- A[12] === B[14]
- A[15] === B[20]

The sequence of calls to addSubMatches will be with the following ranges:
- A[0..4]   /  B[0..6]
- A[5..11]  /  B[7..13]
- A[12..14] /  B[14..19]
- A[15..22] /  B[20..28]

Once addSubMatch begins processing, the first thing it does is match any lines at the beginning and ending of the A and B ranges.  The matches at the beginning are added to the "results" array.  The matches at the end of the range are held, until the disposition of the sandwiched lines are dealt with by determining if this remaining subsequence has any unique lines in common between arrays A and B, in which case a recursive call is made to the core recurseLCS function.  If there are no unique lines in common, then we're at a point where anything in array A can be included in "results" as having been removed, and anything in array B can be included in "results" as having been added.  If having traveled the recursion path, once the recursion unwinds, any intervening results will have been placed on the "results" array, at which time now addSubMatch can place the matches found at the end of the original subsequence on the "results" array, ensuring the proper order of the results.

In terms of complexity, addSubMatch on its own is linear and therefore O(N), but the occasional recursion in essense is recalculating uniqueCommon lines in the smaller subsequence.  It kinda smells like O(N log N), but would have to run a large sample of tests to verify...

### recurseLCS(aLo, aHi, bLo, bHi, uniqueCommonMap) adds values to the "results" array via addSubMatch
Finally we get to the main routine, which basically starts with the entire A and B array ranges, performing the following logic:
- Get the longest common subsequence (LCS) of unique lines for the provided range A[aLo..aHi] and B[bLo..bHi].
- If there are no unique lines, then call addSubMatch with the entire range to add the lines to the "results" array.
- If the LCS did return some unique lines in common for the provided range, then:
  - call addSubMatch with any lines preceding the first LCS entry.
  - Then loop through the LCS entries calling addSubMatch (see explanation in addSubMatch).
  - And finally, call addSubMatch with any lines following the last LCS entry.

Voila!


# References
The patience diff algorithm is credited to Bram Cohen of Bittorrent fame. Additionally, the article by "Alfedenzo" at https://alfedenzo.livejournal.com/170301.html was of immense help in understanding the Longest Common Subsequence (LCS) algorithm, and by tradition, is also the source of the data embedded in the associated example HTML document.

# Example
Simply download the PatienceDiff.js file which contains the complete algorithm, and PatienceDiff.html which exemplifies how to use the algorithm.  Then open PatienceDiff.html in a browser, and press the "=> Diff =>" button to calculate the difference between the two blocks of text.


