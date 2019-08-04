//# helloworld
//my first
#include <stdio.h>
void test(){
	printf("test\n");
}
void main (void){
  printf("Hello World!hhhhhh\n");
}

# -*- coding: UTF-8 -*-

import re
import os

INDENT_STR = '  '
HEADER_STR = '|--'
ROOT_PATH = 'J:\\ProgramGraph\\Lua-2\\'  # 'J:\\ProgramGraph\\Lua-2\\html\\'
node_id_map = {}  # id-name
node_rel_list = []  # tuple(id1 -> id2)
level_stk = []  # id-level stack


def parse_dot_file(file_name):
    """Parsing the file_name dot file and then add item to node_id_map and node_rel_list"""
    dot_file = open(ROOT_PATH + file_name)
    lines = dot_file.readlines()
    node_patt = re.compile(r'\s\sNode(\d+)\s\[label="([\w|_|\\]+)')
    relation_patt = re.compile(r'\s\sNode(\d+)\s->\sNode(\d+)')

    for line in lines:
        res = re.match(node_patt, line)
        if res is not None:
            get_node_id(res)
        else:
            res = re.match(relation_patt, line)
            if res is not None:
                get_node_relation(res)


def get_node_id(res):
    """Generating id-name map according to the result matched by regular expression"""
    node_id = res.group(1)
    node_name = res.group(2)
    node_name = re.sub(r'\\l', '', node_name)
    node_id_map[node_id] = node_name


def get_node_relation(res):
    """Generating id-id list according to the result matched by regular expression"""
    node_id1 = res.group(1)
    node_id2 = res.group(2)
    node_rel_list.append((node_id1, node_id2))


def is_dot_file(file_name):
    """Return True if the file_name is a dot file"""
    pat = re.compile(r'.+\.dot')
    res = re.match(pat, file_name)
    if res is not None:
        return True
    return False


def is_call_graph(file_name):
    """Return True if the file_name is a call graph or icall graph"""
    pat = re.compile(r'[i_]cgraph\.dot$')
    res = re.search(pat, file_name)
    if res is not None:
        return True
    return False


def print_call_list():
    print(node_id_map[node_rel_list[0][0]])
    for rel in node_rel_list:
        id1 = rel[0]
        id2 = rel[1]
        lev1 = create_left_node_level(int(id1))
        # print(INDENT_STR * lev1)
        # print(HEADER_STR + node_id_map[id1])
        print(INDENT_STR * (lev1 + 1) + HEADER_STR + node_id_map[id2])
        #set_right_node_level(int(rel[1]))


def set_right_node_level(id):
    level_stk.append(id)
    #print(level_stk)


def create_left_node_level(id):
    """If the id is not in the list, append it in the list tail. Else clear the items after the id.
    :return index of the id"""
    if id in level_stk:
        idx = level_stk.index(id)
        level_stk[idx + 1:] = []
        #print(level_stk)
        return idx
    else:
        level_stk.append(id)
        #print(level_stk)
        return len(level_stk) - 1


def generate_call_list():
    file_list = os.listdir(ROOT_PATH)
    for file in file_list:
        if is_dot_file(file) and is_call_graph(file):
            parse_dot_file(file)
            print_call_list()
            node_id_map.clear()
            node_rel_list.clear()
            level_stk.clear()


if __name__ == '__main__':
    generate_call_list()
