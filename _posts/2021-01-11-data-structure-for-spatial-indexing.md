---
layout: post
title: Data Structure for Spatial Indexing
date: 2021-01-11 22:45
summary: Spatial Indexing requires a different data structure than a 1D space
categories: algorithm programming tech java
tags: algorithm programming tech java
---

![Photo by Uriel Soberanes](https://images.unsplash.com/photo-1535223289827-42f1e9919769?ixlib=rb-1.2.1&ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&auto=format&fit=crop&w=668&q=80)

Data structure helps us store values within our data and help us efficiently do the operation with those data if we need them. For instance, if we want to store 1-dimensional data points, natural numbers that you will plot in a single line or a string, we can use a 1D array to store these data. To create a fast retrieval (search), we will use a natural order indexing (1 < 2 < 3) or using a data structure like Trie or Binary Tree. 

What if we want to work with the 2D space and store our data to do a fast retrieval? What if we're going to find proximity ordering, such as find all the nearby points that are close to this point?

A natural order indexing will not work since we need to have two different indexes, one for point X and the other for point Y, and we have to search for all the places that are `X + delta` and `Y + delta` in the database and do an intersection.

We will need to use Spatial Indexing.

Spatial Indexing is often used for accessing 2D space efficiently. Use-cases that use Spatial Indexing are:  ride-sharing application (Lyft, Uber), Food delivery service (Door dash) which needs to find the nearest food deliver to, Yelp wants to let you know the nearest restaurant from your location, hit detection, and more.

A couple of Spatial Indexing applications include finding the K nearest neighbor - an application that needs to get the nearest neighbor from the target object.


Range Query: finding an object containing a given point (point query) or overlapping with an area of interest (window of the query).

Spatial Join: Finding pairs of the object that interact spatially with each other. Using intersection, adjacency, and containment of spatial predicates to perform a spatial join.


Now that you get a glimpse of what spatial Indexing is, let's talk about what sort of data structure, we need to store these data points for fast retrieval. If you think it is QuadTree, then you are correct. In the section below, I will explain what a QuadTree is and how it is useful to store sparse data for searching.


## What is a QuadTree?
Quadtrees are a way to partition space so that it is easier to traverse and search. It is a tree data structure that divides the value into four children, quad. A leaf node can hold some values depending on what application you are implementing. The subdivided region can be square or rectangle. A quadTree is similar to a Trie, except that they only have four children, and the way of determining those four children is with some criteria, such as if this point is in a particular range, traverse the top left quadrant.

Quadtree can help you anytime when you need to store sparse data that you need to search. It keeps data particles in the chemical reaction, pixels (image processing), and more.

For this article, I will implement the region quadtree.

Before we start, there are three components in the QuadTree. One is the point object you need to store. In this case, we can do `x` and `y`. Second is the QuadNode, which is the node that you want to hold inside your QuadTree. Lastly is the Tree itself.

## Point class
<script src="https://gist.github.com/edwardGunawan/13cda621214dc5bea3ba2c4e33c1468d.js"></script>

## QuadNode Class

<script src="https://gist.github.com/edwardGunawan/2a05a75f7961820cb1406bf7284cb0be.js"></script>

## QuadTree Class

<script src="https://gist.github.com/edwardGunawan/baaf3ea48942747b73e0c7e28f4227ff.js"></script>

## Insertion
Like the Binary Search tree, we will need to start from the root and check which region the point belongs to. Then, we can recursively traverse down that region until we hit a leaf node.

Then, we insert the point in the quadtree's leaf node and check if the quadtree needs to further `subdivide`. If it needs to promote `subdivide`, we will `subdivide` the region into four quadrants and redistribute the value inside to its children.


<script src="https://gist.github.com/edwardGunawan/56513d0533f2f3dcc42cf02e8bce2dee.js"></script>

## Search
Start from the root, check which region the point belongs to. Recursively traverse down that region until we reach the leaf node. Return the value of that leaf node, which contains a list of points.

<script src="https://gist.github.com/edwardGunawan/dbf63e8672968b0138f7d7a9f8bb1387.js"></script>



## Conclusion
- To store geolocation, a sequential search with natural ordering is not fast enough. We will use spatial Indexing to search for a 2D space. 
- QuadTree is like an equivalent data structure for a binary tree in the 1D space. However, we can use QuadTree for any time you have sparse data that you need to search.
