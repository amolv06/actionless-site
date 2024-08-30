+++
title = "Advanced Filtering of Chess Databases with pgn-extract"
date = 2024-08-29T00:00:00-05:00
tags = ["chess"]
draft = false
+++

## Introduction {#introduction}

Serious Chess players often rely on databases containing millions of games to enhance their studies. These databases are invaluable for studying openings, where a user can enter an exact sequence of moves to reach the position they want to analyze. Chess databases can filter based on such opening sequences with ease. However, when it comes to filtering based on criteria outside of the opening, such as games where there are more than eight minor pieces on the board, games that end in Anastasia's mate, or games where advanced players tackle difficult rook and pawn endings your typical database falls short without external tools.

[pgn-extract](https://www.cs.kent.ac.uk/people/staff/djb/pgn-extract/) is a powerful command-line tool that complements Chess databases by enabling you to filter games based on a wide variety of criteria. In my experience, it is considerably faster than the alternatives. You can download pgn-extract either as a Windows binary or from your Linux distro's repository. Furthermore, it is open-source so if it's not readily available for your platform you can compile the code yourself. In this blog post, we will provide a high-level overview of how pgn-extract works and demonstrate how to use it to filter games that match the examples mentioned earlier.


## pgn-extract {#pgn-extract}

pgn-extract takes a series of flags followed by a list of pgn files as arguments.

```nil
pgn-extract [flags] [pgn-files]
```

Each pgn file can contain multiple games. The flags allow you to define the criteria a game must meet, and if a game matches that criteria it is output to the terminal by default (though the --output flag allows you to specify an output file for the filtered games). pgn-extract offers a wide range of flags, but I'll focus on some of the ones I've found most useful here. For a complete list of features, refer to the [full documentation](https://www.cs.kent.ac.uk/people/staff/djb/pgn-extract/help.html).


## Examples {#examples}

Lichess offers all of its games as [free downloads](https://database.lichess.org/). For the purpose of these examples I am going to use the July 2024 database. You'll need to decompress the downloaded file. On Linux you can use unzstd. On Windows, Lichess recommends using a program called [PeaZip](https://peazip.github.io/). Note that the unzipped file will be around 200GB, so if you're following along make sure that you have the requisite hard disk space.


### Example 1: More than 8 minor pieces {#example-1-more-than-8-minor-pieces}

In this first example we're going to search for games where there are more than eight minor pieces (knights + bishops) on the board at any point during the game. To achieve this, we will use the -z flag, which takes a filename as an argument. This file is specified in a special syntax defining the material balance you want to search for.

At the most basic level, the file will contain lines having two sequences of characters separated by a space, representing the material you want to search for, e.g.

```text
nb rp
```

This will search for positions where one side had a knight and a bishop and the other side had a rook and a pawn. You do not need to specify the king unless it is the only piece one side has.

Note that with the -z flag pgn-extract searches for positions in a color-blind manner. It will output games where either side has the specified material, meaning it doesn't differentiate whether white or black has the knight and bishop versus the rook and pawn. There is also a -y flag, which will only look for positions where white has the material represented by the first sequence of characters and black has the second.

When specifying material you may append certain symbols to a character that specifies a piece, giving it additional meaning:

| Character | Meaning                              |
|-----------|--------------------------------------|
| \*        | Zero or more of that piece           |
| +         | One or more of that piece            |
| d         | Exactly d occurrences of that piece  |
| d+        | d or more occurrences of that piece  |
| d-        | d or fewer occurrences of that piece |

pgn-extract conveniently offers the character 'l' as a way to represent a minor piece, matching either a bishop or a knight. We can now create the file to filter games with more than eight minor pieces. I'll call my file `filter`, but you can name yours whatever you like. Here's the content of the file:

```text
q*r*p*l4 q*r*p*l5+
q*r*p*l3 q*r*p*l6+
q*r*p*l2 q*r*p*l7+
q*r*p*l q*r*p*l8+
q*r*p* q*r*p*l9+
```

The first line looks for games that contain a position where one side has four minor pieces and the other side has 5, regardless of the number of queens, rooks, and pawns on the board. Subsequent lines follow the same pattern, changing only the number of minor pieces each side has. pgn-extract will output any game matching any one of those conditions.

pgn-extract can now be invoked.

```nil
pgn-extract --output matches.pgn -zfilter lichess_db_standard_rated_2024-07.pgn
```

Before diving into the results, it's worth mentioning that pgn-extract filtered through over 90 million games in just over 2 hours. That is a whopping 12,000 games per second on fairly modest hardware.[^fn:1]

pgn-extract found 95 games matching the specified criteria, meaning that games with nine or more minor pieces on the board occur at a frequency of around one in a million. However, most of these 95 games aren't particularly noteworthy. The majority of them appear to be pre-arranged matches or instances of showboating, such as promoting all pawns to knights in an overwhelmingly winning endgame, or unnecessary underpromotions in clearly superior positions. If you've followed along with this post, pgn-extract should have created a file called `match.pgn` containing the pgn of each of the matched 95 games, and no others.

Out of the 95 matches, I identified at least nine interesting games with nine or more minor pieces on the board. The majority of these games stemmed from an opening trap I had not seen before. An example of such a game is embedded below.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

<iframe src="https://lichess.org/embed/game/GwWkMBxq?theme=newspaper&bg=light"
width=600 height=397 frameborder=0></iframe>

</div>

Identical or very similar sequences of moves happened in the games listed below.

-   <https://lichess.org/dFYgGKRe>
-   <https://lichess.org/D2uKBHjY>
-   <https://lichess.org/1JTzWYuY>
-   <https://lichess.org/aZ2u9534>
-   <https://lichess.org/ZI2UB9Wx>

The following three games also featured nine minor pieces on the board, but did not stem from the same opening trap.

-   <https://lichess.org/kpaYfpPy>
-   <https://lichess.org/JgVfvth5>
-   <https://lichess.org/MomxYAy4>


### Example 2: Anastasia's Mate {#example-2-anastasia-s-mate}

pgn-extract cannot, by itself, find games where Anastasia's Mate has been executed. However, I want to highlight a powerful flag that might help us identify positions where Anastasia's mate is possible.

The Anastasia's mating pattern involves either a knight and rook or a knight and queen giving checkmate to a king trapped against the edge of the board on one side and a friendly pawn on the other. In its barest form, it looks like:

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

<iframe width="600" height="371" src="https://lichess.org/study/embed/o8bQslAB/hqri6OkU?theme=newspaper&bg=light" frameborder=0></iframe>

</div>

To find such potential positions we're going to use an FEN pattern search, available through pgn-extract with the -t flag. The text representing a game in a PGN file consists of two parts: "tag pairs" containing metadata about the game as key-value pairs, and "movetext". The -t flag allows you to provide a file as an argument containing tag-based conditions to filter on. However, beyond filtering on just tag-based conditions, you can also filter on whether or not a position is achieved in a game.

Chess positions can be represented using [Forsyth-Edwards Notation (FEN)](https://en.wikipedia.org/wiki/Forsyth%e2%80%93Edwards_Notation), and if you're unfamiliar with it I would advise you to take a look at the link. A position can be filtered for as follows:

```text
FEN "rnbqkbnr/pp1ppppp/8/2p5/4P3/8/PPPP1PPP/RNBQKBNR w KQkq c6 0 2"
```

This would look for positions where `1. e4 c5` has been played from the starting position.

pgn-extract offers an even more versatile option for finding positions with the pseudotag FENPattern, which allows the use of metacharacters in the FEN string to fuzzy match a board. When using FENPattern, you only input the part of the FEN string that specifies the board position and omit additional details. The available metacharacters are

| Character | Meaning                                               |
|-----------|-------------------------------------------------------|
| ?         | Match any square, occupied or unoccupied.             |
| !         | Match any occupied square.                            |
| A         | Match a square occupied by a white piece.             |
| a         | Match a square occupied by a black piece.             |
| \*        | Match zero or more square, occupied or unoccupied     |
| [xyz]     | Match any of the characters represented by x, y, or z |
| [^xyz]    | Match any character not represented by x, y, or z     |

We will use use FENPattern pseudotag to search the Lichess database for the skeleton of the Anastasia mate given in the diagram above, with the black king on h7, a black pawn on g7, a white knight on e7 and a white rook or queen on h1-h5. We do so by adding the content below to the file that defines out filter, which I've called filter2. Note that when using the -t flag, generally multiple instances of the same tag (or pseudotag in our case) are OR-ed together. Different tags are AND-ed together.

```text
FENPattern "*/*N?pk/*1/*[QR]/*/*/*/*"
FENPattern "*/*N?pk/*1/*1/*[QR]/*/*/*"
FENPattern "*/*N?pk/*1/*1/*1/*[QR]/*/*"
FENPattern "*/*N?pk/*1/*1/*1/*1/*[QR]/*"
FENPattern "*/*N?pk/*1/*1/*1/*1/*1/*[QR]"
```

Now that we've set up our file, let's break down the pseudoFEN string on the first line in detail. It asks pgn-extract to look for a board where it ignores the eighth rank (∗); looks for a seventh with a white knight on the e-file, anything on the f-file, a pawn on the g-file, and a king on the h-file (∗N?pk); ignores the sixth rank except ensuring that the h-file is open (∗1); looks for a white queen or rook on the fifth rank (∗[QR]), and ignores the rest of the board (∗/∗/∗/∗). The second line does much of the same as the first, except it ensure that the h-file on both the fifth and sixth ranks is open, and looks for a queen or rook on the h-file of the fourth rank. The rest of the lines simply move the queen or rook back one rank. All of those lines are OR-ed together, so the file basically looks for any game with the Anastasia setup with the queen or rook anywhere from h1-h5.

Invoking this will take filter2 as your filter file and return a file called matches2.pgn. You only need to let it run for a few seconds and can kill the process afterwards, as you will already have several matches.

```nil
pgn-extract -t filter2 --output matches2.pgn lichess_db_standard_rated_2024-07.pgn
```

This will not capture every possible example of Anastasia's mate. There are other configurations that allow Anastasia's mate, which my filter does not look for. And there will be games this search flags where Anastasia's mate is not played -- for example, in cases where there is a piece that can block the check. Nevertheless, this query will find many examples where Anastasia's mate is played, such as [this one](https://lichess.org/m0Ev6DYz#39). The ability to search Chess databases in this manner greatly expands their usefulness.


### Example 3: How do advanced players tackle rook and pawn endings? {#example-3-how-do-advanced-players-tackle-rook-and-pawn-endings}

To find games where advanced players engage in rook and pawn endings, we will once again use the -z and -t flags, though in a slightly different manner than before. Additionally, we will employ the --minmoves flag to ensure that game have a minimum length, increasing the likelihood of finding more interesting endgames.

Previously, we discussed how the -z flag allows us to use special metacharacters appended to piece-representing characters which specify the number of such pieces we expect to find. While the previous metacharacters specified something about the absolute number of pieces a side should have, there are additional metacharacters that allow us to specify the number of pieces in relation to how many pieces the other side has.

| Character | Meaning                                                                |
|-----------|------------------------------------------------------------------------|
| =         | The number of these pieces must be the same as the opponent            |
| #         | The number of these pieces bust be different from the opponent's       |
| &gt;      | The number of these pieces must be more than the opponent has          |
| &lt;      | The number of these pieces must be less than the opponent has          |
| d&gt;     | The number of these pieces must be at least d more than the opponent's |
| d&lt;     | The number of these pieces must be at least d less than the opponent's |
| d&gt;=    | The number of these pieces must be exactly d more than the opponent's  |
| d&lt;=    | The number of these pieces must be exactly d less than the opponent's  |

To keep things interesting, we will look for rook and pawn endgames with an imbalanced number of pawns, where each side has at least one pawn (and both sides have a rook). We will pass filter3 to the -z flag, given below.

```text
rp+ rp>
```

Since we want to study games played by advanced players, we will filter for games where both players are at least 2500 in strength. Additionally, to ensure we're looking for quality games we will filter for games that last at least 5 minutes (300 seconds). We create a file called filter4 containing these criteria:

```text
TimeControl > "300"
WhiteElo >= "2500"
BlackElo >= "2500"
```

With the additional requirement that games be at least 50 moves long, the final command is as follows:

```nil
pgn-extract -z filter3 -t filter4 --minmoves 50 --output match3.pgn lichess_db_standard_rated_2024-07.pgn
```

This finds several candidates. Here is an example:

<iframe src="https://lichess.org/embed/game/0JOb4pHi?theme=newspaper&bg=light"
width=600 height=397 frameborder=0></iframe>


## Conclusion {#conclusion}

pgn-extract has far more functionality than we've shown here. Aside from just filtering Chess positions, it can also manipulate the pgn text en masse. There are filters we haven't looked at that are extremely powerful. One in particular that I like, but that we did not cover here, is the ability to fuzzy match on opening sequences. Even the tags that we explored in this post have more functionality than we could cover in this brief overview, so I encourage you to delve into pgn-extract's complete documentation. Nevertheless, we've seen some pretty cool things pgn-extract can do, and I believe it can be a powerful tool in your Chess studies. Please let me know if you have some other cool ways you've used this tool, as I'd love to learn more myself.

[^fn:1]: We can compare pgn-extract with the speed of [python-chess](https://python-chess.readthedocs.io/en/latest/) as a reference.

    ```python
    import time
    import chess.pgn

    start_time = time.time()
    pgn = open("./lichess_db_standard_rated_2024-07.pgn")

    g = 0
    while g<100000:
        g = g+1
        chess.pgn.read_game(pgn)

    print(time.time() - start_time)
    ```

    This does nothing other than read the game -- no processing -- and it still takes 138 seconds to run. This works out to a rate of 725 games per second. pgn-extract filters and everything, and is still nearly 20x faster than python-chess.
