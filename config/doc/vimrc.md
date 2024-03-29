```
let g:for_c=0

set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
" Git plugin not hosted on GitHub
"Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
"Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Avoid a name conflict with L9
" Plugin 'user/L9', {'name': 'newL9'}




" 888888888888888888888888888888


" " Displays tags in a window, ordered by class etc, i used it instead of
" taglist
Bundle 'majutsushi/tagbar'
"
"

Bundle 'genutils'
"Bundle 'lookupfile'
"
"
" " Fuzzy file, buffer, mru, tag, ... finder with regexp support.
Bundle 'kien/ctrlp.vim'
"
"
" " Fast file navigation
" Bundle 'wincent/Command-T'
"
"
" " Preview the definition of variables or functions in a preview window
" Bundle 'autopreview'
"
"
" " Echo the function declaration in the command line for C/C++
" Bundle 'echofunc.vim'
"
"
Bundle 'FuzzyFinder'
Bundle 'grep.vim'
Bundle 'a.vim'
Bundle 'taglist.vim'
Bundle 'The-NERD-Commenter'
Bundle 'The-NERD-tree'

Bundle 'wombat256.vim'

Bundle 'vim-ruby/vim-ruby'
"
" " Under linux need exec 'dos2unix
" ~/.vim/bundle/QFixToggle/plugin/qfixtoggle.vim'
" Bundle 'QFixToggle'
"
"
" Bundle 'Color-Sampler-Pack'
" Bundle 'altercation/vim-colors-solarized'
" Bundle 'txt.vim'
" Bundle 'mru.vim'
" Bundle 'YankRing.vim'
" Bundle 'tpope/vim-surround.git'
" Bundle 'DoxygenToolkit.vim'
" Bundle 'headerGatesAdd.vim'
" Bundle 'ShowMarks'
" Bundle 'Lokaltog/vim-powerline'
"
"
" " Deal with pairs of punctuations such as (), [], {}, and so on
"Bundle 'kana/vim-smartinput'
"Bundle "MarcWeber/vim-addon-mw-utils"
"Bundle "tomtom/tlib_vim"
"Bundle "honza/snipmate-snippets" 不能使用

"Bundle "garbas/vim-snipmate"

"for coffeescript
Bundle 'kchmck/vim-coffee-script'
















" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList          - list configured plugins
" :PluginInstall(!)    - install (update) plugins
" :PluginSearch(!) foo - search (or refresh cache first) for foo
" :PluginClean(!)      - confirm (or auto-approve) removal of unused plugins
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this lineßß






















































if(has("win32") || has("win95") || has("win64") || has("win16"))
    let g:isWin=1
else
    let g:isWin=0
endif

if(has("mac"))
    let g:isMac=1
else
    let g:isMac=0
endif
set helplang=cn "中文help


map <silent> <C-s> :w<cr>
imap <silent> <C-s> <esc>:w<cr>"


map <F2> :NERDTreeToggle<CR>
let g:NERDTree_title = "[NERD Tree]"
let NERDTreeShowBookmarks=1 "一直显示书签
let NERDTreeChDirMode=2 "
            
let g:neocomplcache_enable_at_startup = 1

set makeprg=gcc\ -Wall\ -o%.o\ %


" Encoding related
set fileencodings=utf-8,gb2312,gbk,gb18030
set termencoding=utf-8

set nocompatible
set autoread<
syntax enable
set hlsearch
set wrap
set nu

if g:for_c==1
  set ts=4
  set expandtab
  set autoindent
  set tabstop=4
  set softtabstop=4
  set shiftwidth=4
else
  set ts=2
  set expandtab
  set autoindent
  set tabstop=2
  set softtabstop=2
  set shiftwidth=2
endif


if has("gui_running")
colorscheme desertEx
else
"colorscheme desert

 colorscheme  wombat256mod "256暗色有看着比较舒服

endif


set t_Co=256 "要使用wobat256mod必须用256色

set cpt=.,w,b
set ignorecase


set wildmenu
set smarttab


map! <c-tab> <tab>


set nobackup
set noswapfile


map <silent> <C-n> :tabnew<cr>
map <silent> <C-a> gg0vG$<cr>
map <silent> qq :tabclose<cr>
map <silent> qb :bd<cr>




let mapleader=","
map <silent> <leader>l :call NERDComment(0, "toggle")<cr> 
map <silent> <leader>k :call NERDComment(0, "sexy")<cr> 
map <silent> <leader>p :tabp<cr> 
map <silent> <leader>n :tabn<cr> 
map <silent> <leader>e :edit ~/.vimrc<cr> 
map <silent> <leader>1 gg<cr> 
map <silent> <leader>g GG<cr> 


" change set nonu map
map <leader>tn :call Toggle_Number()<cr>
function! Toggle_Number()
    if !exists("b:togglenum")
        let b:togglenum=1
    endif
    if b:togglenum==1
        execute "set nonu"
        let b:togglenum=0
    else
        let b:togglenum=1
        execute "set nu"
    endif
endfunction


"Key map for FuzzyFinder                                                       
let mapleader=","
map <leader>ff :FufFile<cr>
map <leader>fb :FufBuffer<cr>


" CTRL-X and SHIFT-Del are Cut
vnoremap <C-X> "+x
vnoremap <S-Del> "+x




" CTRL-C and CTRL-Insert are Copy
vnoremap <C-C> "+y
vnoremap <C-Insert> "+y

" CTRL-C and CTRL-Insert are Copy
if g:isMac == 1
    nmap <C-V> :set paste<CR>:r !pbpaste<CR>:set nopaste<CR>
    imap <C-V> <Esc>:set paste<CR>:r !pbpaste<CR>:set nopaste<CR>
    nmap <C-C> :.w !pbcopy<CR><CR>
    vmap <C-C> :w !pbcopy<CR><CR>
endif


" CTRL-V and SHIFT-Insert are Paste
map <C-V> "+gP


cmap <C-V> <C-R>+


" Use CTRL-Q to do what CTRL-V used to do
noremap <leader>v <C-V>









" &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&


let mapleader= ","

" my configure
set noimdisable
set iminsert=0
set imsearch=0
set noswapfile
" config it for change buffer without save it when changed
set hidden "in order to switch between buffers with unsaved change
map <silent><F8> :NERDTree<CR>

set statusline=%<%F%1*%m%*%r%y%=%b\ 0x%B\ \ [l,c]%l,%c%V\ %p%%\ %{fugitive#statusline()}


let Tlist_WinHeight=25
let Tlist_File_Fold_Auto_Close=1
map <silent><F4> :ts<cr>

" 设置Tlist
let Tlist_Ctags_Cmd = "/usr/bin/ctags"
let Tlist_Use_Right_Window=1
let Tlist_File_Fold_Auto_Close=1
let Tlist_Show_One_File=1
let Tlist_GainFocus_On_ToggleOpen=1
let Tlist_Exit_OnlyWindow=1
let Tlist_Show_Menu=1
let Tlist_Process_File_Always=1
map <silent><F9> :TlistToggle<cr>

" 设置quickfix
nmap cn :cn<cr>
nmap cp :cp<cr>
nmap cw :cw 10<cr>



" 在 vim 启动的时候默认开启 NERDTree（autocmd 可以缩写为 au）
autocmd VimEnter * NERDTree
" autocmd BufWrite * execute ":s/\s*$//" :%s/\s*$//g
" autocmd BufWrite * execute ":%s/\s*$//g"

" 当打开 NERDTree 窗口时，自动显示 Bookmarks
let NERDTreeShowBookmarks=1

set nu
map <tab> :tabn<CR>
let g:user_zen_settings = {
      \  'indentation' : '  '
      \}
let g:indent_guides_guide_size = 1

" hightlight col and line
set cursorline
"set cursorcolumn

if has("gui_running")
  colorscheme desert
  set bs=2
  set ruler
  set gfn=Monaco:h16
  set shell=/bin/bash
endif


"let mapleader=","
let mapleader=" "
let g:ctrlp_map = '<leader>p'
let g:ctrlp_cmd = 'CtrlP'
map <leader>f :CtrlPMRU<CR>
let g:ctrlp_custom_ignore = {
    \ 'dir':  '\v[\/]\.(git|hg|svn|rvm)$',
    \ 'file': '\v\.(exe|so|dll|zip|tar|tar.gz|pyc)$',
    \ }
let g:ctrlp_working_path_mode=0
let g:ctrlp_match_window_bottom=1
let g:ctrlp_max_height=15
let g:ctrlp_match_window_reversed=0
let g:ctrlp_mruf_max=500
let g:ctrlp_follow_symlinks=1



nmap ] <C-O>


" normal模式下，ct触发ctags索引建立
nmap ct :!/usr/bin/ctags -R .<CR> 1
nnoremap <F2> :grep -rn --exclude-dir=log --exclude=tags --exclude-dir=.git --exclude-dir=xml --exclude-dir=plist --exclude-dir=bin --exclude-dir=third_party --exclude-dir=public --exclude-dir=tmp <C-R><C-W> --exclude=*.o --exclude=*.out --exclude=.* .<CR>
nnoremap <F3> :grep -rn --exclude-dir=log --exclude=tags --exclude-dir=.git --exclude-dir=xml --exclude-dir=plist --exclude-dir=bin --exclude-dir=third_party --exclude-dir=public --exclude=*.o --exclude-dir=tmp --exclude=*.out --exclude=.*

let NERDTreeIgnore=['\.vim$', '\~$', '.*\.o$', '.*\.a$', 'bserver$', 'gclient$', 'gserver$', 'install_pack$', '^tags$']


```