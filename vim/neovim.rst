========================================
Neovim
========================================

Neovim 是從 Vim 7.4.160 fork 出去的專案，
目標是要重構 Vim、改善架構、讓大家更容易貢獻等等，
而在那之後 Neovim 依然有儘量把 Vim 的 patches porting 到 Neovim 跟著改善。

一些改善：

* source code 整理
* 改用 CMake
* 把原本的 C code porting 前往 C99 (不知道以後可否變 C11 XD?)
* 使用 libuv 取代原本一些針對特定 OS 的 code
* 支援使用新的 msgpack API 來做 RPC (remote procedure call)
* 清除大量的 memory 相關問題
* 接上 Continuous Integration 為每次 commit 做測試
* 加上 AddressSanitizer & Memory Sanitizer 幫助測試
* ...

----

Neovim 裡的 runtime files (例如 syntax highlight) 還是使用跟 Vim 一樣的內容，
目前沒有要另外維護一份，
所以要更動的話直接送到 upstream Vim，
Neovim 會定期 update 過來。

----

某篇晚上發現 neovim 有 vim patches 在 neovim 的 `狀況表 <http://neovim.io/doc/reports/vimpatch/>`_ ，
有的 patch neovim 不需要，有的需要修改，
之後有空要來幫忙 :P

其中一個目標是 `patch 7.4.754 <https://github.com/vim/vim/commit/v7-4-799>`_ ，
Visual mode 的 control+a 來增加數字、control+x 減數字。


Build From Source
========================================

Build with Sanitizer
------------------------------

.. code-block:: sh

    $ CC=clang cmake .. -DCLANG_ASAN_UBSAN=ON


Run Test
------------------------------

Run Test With Valgrind
------------------------------

.. code-block:: sh

    $ VALGRIND=1 make test



Porting Vim patches to Neovim
========================================

Neovim 內建有個 script 可以幫助 porting Vim patches 到 Neovim，
假設想 porting 的是 patch "7.4.123" :

.. code-block:: sh

    ./scripts/vim-patch.sh 7.4.123

接下來可能會有 merge conflict 要修。


修改的注意事項
------------------------------

Test Cases
++++++++++++++++++++

如果有在 ``src/testdir/test*.in`` 加入新的 test 的話，
用 script 轉成 lua 版本 (轉完可能要修一下)：

.. code-block:: sh

    ./scripts/legacy2luatest.pl src/nvim/testdir/test_writefile.in test/functional/legacy


Code Differences
++++++++++++++++++++

+-------------------------------------+-------------+
| Deprecated or removed               | Replacement |
+=====================================+=============+
| vim_free                            | xfree       |
+-------------------------------------+-------------+
| malloc, alloc, lalloc               | xmalloc     |
+-------------------------------------+-------------+
| calloc                              | xcalloc     |
+-------------------------------------+-------------+
| realloc, vim_realloc                | xrealloc    |
+-------------------------------------+-------------+
| mch_memmove                         | memmove     |
+-------------------------------------+-------------+
| vim_memset, copy_chars, copy_spaces | memset      |
+-------------------------------------+-------------+
| vim_strncpy, strncpy, strcpy        | xstrlcpy    |
+-------------------------------------+-------------+

+-----------+---------------------+-------------------------------+
| Data type | Format (Vim source) | Portable format (Nvim source) |
+===========+=====================+===============================+
| long      | "%ld"               | "%" PRId64                    |
+-----------+---------------------+-------------------------------+
| size_t    | "%ld"               | "%zu"                         |
+-----------+---------------------+-------------------------------+


PR 格式
------------------------------

* PR 標題要有 ``vim-patch:7.4.xxx``
* commit message
    - 第一行為 "vim-patch:7.4.xxx"
    - 接著空一行
    - 問題 "Problem:    Accessing memory before an allocated block"
    - 解法 "Solution:   Check for not going before the start of a pattern.  (Dominique Pelle)"
    - 空一行
    - 原本 patch 的 link
    - ``vim-patch.sh`` 這隻 script 可以幫忙


commit message 範例 :

::

    vim-patch:7.4.799

    Problem:    Accessing memory before an allocated block.
    Solution:   Check for not going before the start of a pattern.  (Dominique Pelle)

    https://github.com/vim/vim/commit/v7-4-799


Example
------------------------------

Example 1
++++++++++++++++++++

* `原 patch <https://github.com/vim/vim/commit/v7-4-492>`_
* `porting 後的 patch <https://github.com/neovim/neovim/commit/1d5222985ccad4cd31c4b7498810ff2968dd33a2>`_

Example 2
++++++++++++++++++++

* `幫忙為 Vim patches 標上 NA <https://github.com/neovim/neovim/pull/2832>`_


其他可能的項目
========================================

* `Question: Autocomplete Improvements <https://www.bountysource.com/issues/1447132-question-autocomplete-improvements>`_
    - 理想情況應該是有個 general 的 complete 程式，各個 editor 做好 API 接上去
    - `Atom 的 Autocomplete <http://blog.atom.io/2015/05/15/new-autocomplete.html>`_

* `Improve large files support <https://www.bountysource.com/issues/1832251-improve-large-files-support>`_
* `Plugin Package Manager <https://www.bountysource.com/issues/1430941-plugin-package-manager>`_

Plugin Manager
========================================

* `vim-plug <https://github.com/junegunn/vim-plug>`_


Plugin
========================================

* `VimAwesome <http://vimawesome.com/>`_

* `Neomake <https://github.com/benekastah/neomake>`_
    - A plugin for asynchronous :make using Neovim's job-control functionality. It is inspired by the excellent vim plugins Syntastic and Dispatch.
    - https://jacky.wtf/weblog/moving-to-neovim/

Challenge
========================================

* `VimGolf - real Vim ninjas count every keystroke! <http://www.vimgolf.com/>`_

Funny
========================================

* `Vim on a Mechanical Typewriter <http://qqrs.github.io/blog/2013/05/03/vim-on-a-mechanical-typewriter/>`_


Idea
========================================

* use Pygement to do syntax hightlight

Reference
========================================

* `Neovim - Merging patches from upstream Vim <https://github.com/neovim/neovim/wiki/Merging-patches-from-upstream-Vim>`_
* `Neovim - Contributing <https://github.com/neovim/neovim/wiki/Contributing>`_
* `[GitHub] neovim/neovim <https://github.com/neovim/neovim>`_
* `[GitHub] vim/vim <https://github.com/vim/vim>`_
* `neovim dev-doc (generated by Doxygen) <http://neovim.io/doc/dev/index.html>`_
* `Vim Hall of WTF <http://geoff.greer.fm/vim/>`_
* `Why Neovim is Better than Vim <http://geoff.greer.fm/2015/01/15/why-neovim-is-better-than-vim/>`_
* `why does VimL suck? <http://www.reddit.com/r/vim/comments/1bf672/why_does_viml_suck/>`_
* `Learn Vimscript the Hard Way <http://learnvimscriptthehardway.stevelosh.com/>`_
