```
多级下拉菜单
https://blog.csdn.net/weixin_34203832/article/details/93864987
```

```

<div class="dropdown open">
    <button class="btn btn-primary dropdown-toggle" type="button" id="dropdownMenu1" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
        Dropdown
    </button>
    <div class="dropdown-menu" aria-labelledby="dropdownMenu1">
        <div class="dropdown-submenu">
            <a class="dropdown-item" tabindex="-1" href="javascript:;">一级菜单</a>
            <div class="dropdown-menu dropdown-" aria-labelledby="dropdownMenu2">
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Action</a>
            </div>
        </div>
        <div class="dropdown-divider"></div>
        <a class="dropdown-item" href="#">Action</a>
        <a class="dropdown-item" href="#">Action</a>
        <a class="dropdown-item" href="#">Action</a>
    </div>
</div>

<br><br><br><br><br><br><br><br><br><br>


<style type="text/css">


        .dropdown-submenu {
            position: relative;
        }
        .dropdown-submenu > .dropdown-menu {
            top: 0;
            left: 100%;
            margin-top: -6px;
            margin-left: -1px;
            -webkit-border-radius: 0 6px 6px 6px;
            -moz-border-radius: 0 6px 6px;
            border-radius: 0 6px 6px 6px;
        }
        .dropdown-submenu:hover > .dropdown-menu {
            display: block;
        }
        .dropdown-submenu > a:after {
            display: block;
            content: " ";
            float: right;
            width: 0;
            height: 0;
            border-color: transparent;
            border-style: solid;
            border-width: 5px 0 5px 5px;
            border-left-color: #ccc;
            margin-top: 5px;
            margin-right: -10px;
        }
        .dropdown-submenu:hover > a:after {
            /*border-left-color: #fff;*/
        }
        .dropdown-submenu.pull-left {
            float: none;
        }
        .dropdown-submenu.pull-left > .dropdown-menu {
            left: -100%;
            margin-left: 10px;
            -webkit-border-radius: 6px 0 6px 6px;
            -moz-border-radius: 6px 0 6px 6px;
            border-radius: 6px 0 6px 6px;
        }
</style>




```