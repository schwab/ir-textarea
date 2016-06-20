# Unwrap/Normalize - Edge selection algorithm

## Detection of the COMMON WRAP
The COMMON WRAP node is the key to defining our unwrap/normalization insertion points. It's the parent that any new node fragements will be added to and from which our selection will be removed to clean up its formatting. Determining it accurately is essential to the operation of the unwrapp algorithm on the edges of the selection. Once we find the COMMON WRAP, we'll be able to create new node tree fragments for the selected part which are de-void of the formatting we need to remove.  

We find the COMMON WRAP of a selection by walking up its ancestor tree until we either get to the first non-custom container (which becomes the COMMON WRAP), or we determine there are no other wraps above it before reaching the non-custom container which would apply the formatting we are attempting to remove. **The COMMON WRAP must not be a node which applies the formatting we are unwrapping.**

Note, *if we find the COMMON WRAP and no other wraps below that point apply the formatting we are attempting to remove, then are done and can exit the edge wrap removal algorithm.*

Once the COMMON WRAP is defined, we can now create the parts (part0, part1, part2) based on this common ancestor.

## The three parts
The 3 parts (part0,part1,part2) include all the leaf nodes under the COMMON WRAP. In order that we can apply the correct treatment to the selection and surrounding leaf nodes the parts are deinfed as : 
..* *part0* - all leaf nodes before (to the left of) the selection that are under the COMMON WRAP
..* *part1* - the actual selection nodes/text
..* part2 - all leaf nodes after the selection that are in the COMMON WRAP

The ancestor's list of each part are needed to perform the manipulations of the dom required to unwrap the part1 selection and maintain the existing formattin of part0 and part2.

## Unwrap/Normalization algorithm
Since the formatting we are attempting to remove from part1 can structurally include other content that we need to maitain and the formatting of part2 must be left intact, we can define a basic algorithm to perform this work now that the definitions are in place.  Briefly, the algorithm is this, 1. remove part1 from the COMMON WRAP
2. remove part2 (if it exists)
3. reinsert part1 into the COMMON WRAP with a node tree devoid of the targetted formatting 
4. reinsert part2 into the COMMON WRAP with all its formatting intact.

Now, let's add some detail to these general steps to more formally define this algorithm : 
1. remove the selection (part1) from the existing node structure (leaving part0 intact) and create a clone of it's node tree (ancestors) back up the node which is a child of the COMMON WRAP. This cloned is called *part1Fragment*
..* normalize/unwrap part1Fragment to eliminate any embedded wraps that pertain only to the content in part1 (the existing method wrap.removeNodeWraps already does this succesfully so we just need to call it on the part1 node fragement)
2. create a clone of the ancestor nodes of part2 called *part2Fragment* in which the top parent node is a clone of the one that was sitting in COMMON WRAP
3. insert part1Fragment into the COMMON WRAP as a nextSibling of the part0 topmost node (the ancestor which contains all the formatting elements of part0)
4. insert part2Fragment into COMMON WRAP as a nextSibling of part1Fragment  

**If part0 does not exist because we selected everything in the COMMON WRAP, then part1Fragment is inserted as the first child of COMMONWRAP **
** If part2 does not exist, the we simple skip the part2 steps **

