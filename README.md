[![Build Status](https://travis-ci.org/ITCase/sqlalchemy_mptt.svg?branch=master)](https://travis-ci.org/ITCase/sqlalchemy_mptt)
[![Coverage Status](https://coveralls.io/repos/ITCase/sqlalchemy_mptt/badge.png)](https://coveralls.io/r/ITCase/sqlalchemy_mptt)
[![Stories in progress](https://badge.waffle.io/itcase/sqlalchemy_mptt.png?label=in progress&title=In Progress)](https://waffle.io/itcase/sqlalchemy_mptt)
[![PyPI](http://img.shields.io/pypi/dm/sqlalchemy_mptt.svg)](https://pypi.python.org/pypi/sqlalchemy_mptt)
[![Bountysource](https://www.bountysource.com/badge/tracker?tracker_id=1148440)](https://www.bountysource.com/trackers/1148440-sqlalchemy_mptt?utm_source=1148440&utm_medium=shield&utm_campaign=TRACKER_BADGE)

Library for implementing Modified Preorder Tree Traversal with your SQLAlchemy Models and working with trees of Model instances, like django-mptt.
Docs http://sqlalchemy-mptt.readthedocs.org/

![Nested sets traversal](https://rawgithub.com/ITCase/sqlalchemy_mptt/master/docs/img/2_sqlalchemy_mptt_traversal.svg)

The nested set model is a particular technique for representing nested sets (also known as trees or hierarchies) in relational databases.

Installing
----------

Install from github:

    pip install git+http://github.com/ITCase/sqlalchemy_mptt.git

PyPi:

    pip install sqlalchemy_mptt

Source:

    python setup.py install

Usage
-----

Add mixin to model

```python
from sqlalchemy import Column, Integer, Boolean
from sqlalchemy.ext.declarative import declarative_base

from sqlalchemy_mptt.mixins import BaseNestedSets

Base = declarative_base()


class Tree(Base, BaseNestedSets):
    __tablename__ = "tree"

    id = Column(Integer, primary_key=True)
    visible = Column(Boolean)

    def __repr__(self):
        return "<Node (%s)>" % self.id

Tree.register_tree()
```
Now you can add, move and delete obj

Insert node
-----------

```python
node = Tree(parent_id=6)
session.add(node)
```

            level           Nested sets example
            1                    1(1)22
                    _______________|___________________
                   |               |                   |
            2    2(2)5           6(4)11             12(7)21
                   |               ^                   ^
            3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                  |          |
            4                                  14(9)15   18(11)19

            level     Insert node with parent_id == 6
            1                    1(1)24
                    _______________|_________________
                   |               |                 |
            2    2(2)5           6(4)13           14(7)23
                   |           ____|____          ___|____
                   |          |         |        |        |
            3    3(3)4      7(5)8    9(6)12  15(8)18   19(10)22
                                       |        |         |
            4                      10(23)11  16(9)17  20(11)21

Delete node
-----------

```python
node = session.query(Tree).filter(Tree.id == 4).one()
session.delete(node)
```

            level           Nested sets example
            1                    1(1)22
                    _______________|___________________
                   |               |                   |
            2    2(2)5           6(4)11             12(7)21
                   |               ^                   ^
            3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                  |          |
            4                                  14(9)15   18(11)19

            level         Delete node == 4
            1                    1(1)16
                    _______________|_____
                   |                     |
            2    2(2)5                 6(7)15
                   |                     ^
            3    3(3)4            7(8)10   11(10)14
                                    |          |
            4                     8(9)9    12(11)13

Update node
-----------

```python
node = session.query(Tree).filter(Tree.id == 8).one()
node.parent_id = 5
session.add(node)
```

            level           Nested sets example
                1                    1(1)22
                        _______________|___________________
                       |               |                   |
                2    2(2)5           6(4)11             12(7)21
                       |               ^                   ^
                3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                      |          |
                4                                  14(9)15   18(11)19

            level               Move 8 - > 5
                1                     1(1)22
                         _______________|__________________
                        |               |                  |
                2     2(2)5           6(4)15            16(7)21
                        |               ^                  |
                3     3(3)4      7(5)12   13(6)14      17(10)20
                                   |                        |
                4                8(8)11                18(11)19
                                   |
                5                9(9)10

Move node (support multitree)
-----------------------------

![Nested sets multitree](https://rawgithub.com/ITCase/sqlalchemy_mptt/master/docs/img/3_sqlalchemy_mptt_multitree.svg)

Move inside

```python
node = session.query(Tree).filter(Tree.id == 4).one()
node.move_inside("15")
```
                     4 -> 15
            level           Nested sets tree1
            1                    1(1)16
                    _______________|_____________________
                   |                                     |
            2    2(2)5                                 6(7)15
                   |                                     ^
            3    3(3)4                            7(8)10   11(10)14
                                                    |          |
            4                                     8(9)9    12(11)13

            level           Nested sets tree2
            1                     1(12)28
                     ________________|_______________________
                    |                |                       |
            2    2(13)5            6(15)17                18(18)27
                   |                 ^                        ^
            3    3(14)4    7(4)12 13(16)14  15(17)16  19(19)22  23(21)26
                             ^                            |         |
            4          8(5)9  10(6)11                 20(20)21  24(22)25


Move after

```python
node = session.query(Tree).filter(Tree.id == 8).one()
node.move_after("5")
```

           level           Nested sets example
                1                    1(1)22
                        _______________|___________________
                       |               |                   |
                2    2(2)5           6(4)11             12(7)21
                       |               ^                   ^
                3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                      |          |
                4                                  14(9)15   18(11)19

            level               Move 8 after 5
                1                     1(1)22
                         _______________|__________________
                        |               |                  |
                2     2(2)5           6(4)15            16(7)21
                        |               ^                  |
                3     3(3)4    7(5)8  9(8)12  13(6)14   17(10)20
                                        |                  |
                4                    10(9)11            18(11)19

Move to top level

```python
node = session.query(Tree).filter(Tree.id == 15).one()
node.move_after("1")
```

            level           tree_id = 1
            1                    1(1)22
                    _______________|___________________
                   |               |                   |
            2    2(2)5           6(4)11             12(7)21
                   |               ^                   ^
            3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                  |          |
            4                                  14(9)15   18(11)19

            level           tree_id = 2
            1                     1(15)6
                                     ^
            2                 2(16)3   4(17)5

            level           tree_id = 3
            1                    1(12)16
                     _______________|
                    |               |
            2    2(13)5          6(18)15
                    |               ^
            3    3(14)4     7(19)10   11(21)14
                               |          |
            4               8(20)9    12(22)13
