import re

start_sym = ""
productions = {}
first_table = {}
follow_table = {}
table = {}

file = '077.txt'
grammar = open(file, "r")


def creatProduction(grammar):
    global start_sym
    for production in grammar:
        lhs, rhs = re.split("->", production)
        rhs = re.split("\||\n", rhs)
        productions[lhs] = set(rhs) - {''}
        if not start_sym:
            start_sym = lhs


def isNonterminal(sym):
    if sym.isupper():
        return True
    else:
        return False


def firstFunc(sym):
    if sym in first_table:
        return first_table[sym]
    if isNonterminal(sym):
        first = set()
        for x in productions[sym]:
            if x == '#':
                first = first.union('#')
            else:
                for i in x:
                    fst = firstFunc(i)
                    if i != x[-1]:
                        first = first.union(fst - {'#'})
                    else:
                        first = first.union(fst)
                    if '#' not in fst:
                        break
        return first
    else:
        return set(sym)


def followFunc(sym):
    if sym not in follow_table:
        follow_table[sym] = set()
    for nt in productions.keys():
        for rule in productions[nt]:
            pos = rule.find(sym)
            if pos != -1:
                if pos == (len(rule) - 1):
                    if nt != sym:
                        follow_table[sym] = follow_table[sym].union(followFunc(nt))
                else:
                    first_next = set()
                    for next in rule[pos + 1:]:
                        fst_next = firstFunc(next)
                        first_next = first_next.union(fst_next - {'#'})
                        if '#' not in fst_next:
                            break
                    if '#' in fst_next:
                        if nt != sym:
                            follow_table[sym] = follow_table[sym].union(followFunc(nt))
                            follow_table[sym] = follow_table[sym].union(first_next) - {'#'}
                    else:
                        follow_table[sym] = follow_table[sym].union(first_next)
    return follow_table[sym]


def printTable():
    print("First")
    for nt in productions:
        print(nt + ":" + str(first_table[nt]))
    print("\n")
    print("Follow")
    for nt in productions:
        print(nt + ":" + str(follow_table[nt]))
    print("\n")


creatProduction(grammar)
for nt in productions:
    first_table[nt] = firstFunc(nt)
follow_table[start_sym] = set('$')
for nt in productions:
    follow_table[nt] = followFunc(nt)

printTable()
