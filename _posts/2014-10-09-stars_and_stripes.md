---
layout: post
title: Stars and Stripes
published: true
---

The first annual summit for the Moore/Sloan Data Science Environments was in full swing on 
our Tuesday incubator day.  Today, I spoke with Andy Becker on the moving object detection project.
The goal is to find sets of pixels in an image database that might correspond to the same moving object,
then add them up and see if the result is above a significance threshold.  A challenge is to find
the pixels that overlap the trajectories when there are millions of images, millions of pixels per image,
and millions of trajectories.

The approach we discussed was to use a spatial index to prune the search space.
PostgreSQL equipped with PostGIS supports efficient topological queries between geometric objects --- overlaps, contains, etc.

To make spatial search efficient, PostGIS provides an R-tree implementation.  An R-tree is not often the fastest multi-dimensional index, but it's pretty insensitive to idiosyncracies in the data.  The idea is to organize the data into a tree of bounding boxes, attempting to minimize overlap between siblings.  Unlike, say, a kd-tree, the data is partitioned instead of the space, which can provide some flexibility for skewed spatial distributions and unusual shapes.

In this case, perhaps the simplest way to use the database is to build a table Image(imgid, timestamp, bbox) and a table Trajectory(trajid, timestamp, point), then find all pairs (imgid, trajid) such that the point in the trajectory is contained in the bbox of the image.  The hope is that this spatial join will significantly prune the search space before trying to find individual pixels.

