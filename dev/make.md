---
layout: page
title: Make
---

# Make

## Debugging

To print the target name and the full list of dependencies, add this to a recipe:

    echo DEBUG: $@: $^

## Target-Specific Variables

Can define different values based on target currently building.
Only available withing the context of a target?s recipe.
See <https://www.gnu.org/software/make/manual/html_node/Target_002dspecific.html>

## Automatic Variables

    $@ target $\< first prerequisite

