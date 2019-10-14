# spf13-vim

    leader is  < Space>

## nerdtree

    c-e : show/hide nerdtree
    < space> e : location current file in tree

## ctrlp

    find file/buffer/function/mru

    c-p: start ctrlp
    F5: refresh cache
    c-f/c-b: switch mode(file/buffer/function/mru)
    c-j/c-k: selected down/up
    c-t: open selected in new tab
    c-v: open selected in split |
    c-x: open selected in split -
    c-o: open selected in split |
    c-z: makr/unmark file, next operation is open selected file
    c-n/c-p: select history cmd in prompt
    c-d: filename search or full path search

## surround

    easy to add/delete with {("")}

    _cs_

    cs"' --> csAB: replace A to B,eg: "ab" to 'ab'
    cs"<p>, eg: "ab" to <p>ab</p>
    cst", eg: <p>ab</p> to "ab"

    _ds_
    ds" --> dsA: delete A pair, eg:{abc} to abc

## nerdcommenter

    <Space>c<Space> comment
    <Space>cc/cu comment to one line

## fugitive

    git in vim

## piv

    for php

## tabularize

    align
    <Space>a=

## Tagbar

    <Space>tt: show/hide tagbar

    ctags -R; in dir of project
    c-]/c-t: jump to declaration and define

## easyMotion

    <Space><Space>w // w mode,
    <Space><Space>b
    <Space><Space>j
    <Space><Space>k
    <Space><Space>s  // search
    <Space><Space>fa  // f mode, search a
