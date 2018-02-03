## Regular Expressions {#regular-expressions}

Regular Expressions stand on the language automaton and formal languages. The idea of Regular Expression (RE) similar to the idea of Knuth Morris Pratt (KMP) Algorithm, but the goal of Regular Expressions is try to find such a given pattern in a string while KMP only capable to find such substring. To do that, we need more abstract machine to capture expressive pattern that support necessary logic such as ‘or’ and repetition. We can construct such abstract machine by implementing **Nondeterministic** **Finite Automaton** (**NFA**) on **Digraph** structure. A term _nondeterministic_ refers that digraph might contains multiple path contrast to the construction of DFA which only has single path.

Similar to arithmetic symbol in mathematic, we can define such similar pattern in the languages as **syntax**. Table below give four patterns with examples:

| Name | Notation | Match |
| --- | --- | --- |
| Concatenation | {A B} | AB |
| Or | A | B | A, B |
| Closure | AB* | A, AB, ABBB |
| Parentheses | C (AC | B) D | CACD, CBD |

Table 1: Language patterns

Another important feature of language is **shortcut** expression. Using a certain shortcut can save us from being type unnecessary long expression. Table below give example of shortcuts:

| Name | Notation | Example | Shortcut for | Match |
| --- | --- | --- | --- | --- |
| Wildcard | . | A . B |  |  |
| Specified Set | _Enclosed in_ [] | [AEIOU]* |  |  |
| Range | Enclosed in [] | [A-Z] |  |  |
| Complement | _Enclosed in_ [] | [^AEIOU]* |  |  |
| At least 1 | + | (AB)+ | (AB)(AB)* | AB ABABAB |
| O or 1 | ? | (AB)? | ‘any’ | AB | ‘any’AB |
| Specific | Count in {} | (AB){3} | (AB)(AB)(AB) | ABABAB |
| Range | Range in {} | (AB){1-2} | (AB) | (AB)(AB) | AB ABAB |

Table 2: Shortcut expression

Building an NFA corresponding to an RE is similar to constructing arithmetic expression using Djikstra’s two-stack algorithm. The differences to Djikstra are:

*   RE does not have an explicit operator for concatenation.

*   RE have a unary operator, for closure (*)

*   RE have only one binary operator, for or (|)

The basic NFA construction consist a digraph G and _epsilon_-transition (transition in edge without scanning text). Below the description of basic rule to construct NFA for RE:

*   **C****oncatenation** is every match transition.

    ![](assets/image1.png)

*   Use stack to handle **Parentheses**: Push each left parenthesis on the stack and pop each time encountered with right parenthesis.

*   Handle **Closure** in two condition: After single character or after right parenthesis.

    ![](assets/image2.png)

![](assets/image3.png)

*   Handle **Or expression** by implementing two _epsilon-_transition.

![](assets/image4.png)

An image below illustrated a full NFA of RE _(.*AB((C|D*E)F)*G)_

![](assets/image5.png)

Below the implementation of NFA of RE in java:

1.  public class NFA

2.  {

3.  private char[] re;// Match transitions

4.  private Digraph G;// Epsilon transitions

5.  private int M;// Number of states

6.  public NFA(String regexp)

7.  {

8.  Stack

    **Illegal HTML tag removed :** ops = new Stack<integer>();</integer>

9.  re = regexp.toCharArray();

10.  M = re.length;

11.  G = new Digraph(M+1);

12.  for (int i = 0; i &lt; M; i++) {

13.  int lp = i;

14.  if (re[i] == &#039;(&#039; || re[i] == &#039;|&#039;)

15.  ops.push(i);

16.  else if (re[i] == &#039;)&#039;) {

17.  int or = ops.pop();

18.  if (re[or] == &#039;|&#039;) {

19.  lp = ops.pop();

20.  G.addEdge(lp, or+1);

21.  G.addEdge(or, i);

22.  } else {

23.  lp = or;

24.  }

25.  }

26.  if (i &lt; M-1 &amp;&amp; re[i+1] == &#039;*&#039;) {

27.  G.addEdge(lp, i+1);

28.  G.addEdge(i+1, lp);

29.  }

30.  if (re[i] == &#039;(&#039; || re[i] == &#039;*&#039; || re[i] == &#039;)&#039;)

31.  G.addEdge(i, i+1);

32.  }

33.  }

34.  public boolean recognizes(String txt)

35.  {

36.  Bag**Illegal HTML tag removed :** pc = new Bag<integer>();</integer>

37.  DirectedDFS dfs = new DirectedDFS(G, 0);

38.  for (int v = 0; v &lt; G.V(); v++)

39.  if (dfs.marked(v)) pc.add(v);

40.  for (int i = 0; i &lt; txt.length(); i++) {

41.  Bag**Illegal HTML tag removed :** match = new Bag<integer>();</integer>

42.  for (int v : pc)

43.  if (v &lt; M)

44.  if (re[v] == txt.charAt(i) || re[v] == &#039;.&#039;)

45.  match.add(v+1);

46.  pc = new Bag**Illegal HTML tag removed :** ();

47.  dfs = new DirectedDFS(G, match);

48.  for (int v = 0; v &lt; G.V(); v++)

49.  if (dfs.marked(v)) pc.add(v);

50.  }

51.  for (int v : pc)

52.  if (v == M) return true;

53.  return false;

54.  }

55.  }

The critical transition of constructing NFA is when we dealing with right parenthesis. Similar to Djikstra two-stack, we need to store every meta character “*, |, (” into stack. When hit right parenthesis character ‘)’, we popped out item in the stack, but if popped out character is ‘|’, we need extra handling to do _or_ _operation_ which is has two _epsilon_-transition.

The total running time of our implementation of Regular Expression using NFA would be **MN** which is M time for constructing NFA and N time for doing scanning over text.