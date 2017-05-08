---
layout: page
title: Unit Tests
---

## Naming Strategy

http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html

    UnitOfWork_StateUnderTest_ExpectedBehavior
    Foo_BadParam1_ShouldThrowException()
    AverageColor_AllBlackPixels_ShouldReturnBlackAverage

## Google Test

Your project needs to build libgtest.a with your compiler settings and link to it.
For example, add this section to your makefile:

    # Clone googletest from here: https://github.com/google/googletest.git
    GTEST_DIR=/home/smeredith/src/googletest/googletest

    a.out : main.o libgtest.a
        g++ --std=c++11 main.o libgtest.a -lpthread

    main.o : main.cpp BigInt.h
        g++ --std=c++11 -Wall -I ${GTEST_DIR}/include -c main.cpp

    libgtest.a : gtest-all.o
        ar -rv libgtest.a gtest-all.o

    gtest-all.o :
        g++ --std=c++11 -I ${GTEST_DIR}/include -I ${GTEST_DIR} -c ${GTEST_DIR}/src/gtest-all.cc

This will generates the libraries libgtest.a (which contains GTest binaries).
You need to link that with your project.
