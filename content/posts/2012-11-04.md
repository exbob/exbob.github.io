---
title: 《Unix Shell 范例精解》第五章 sed 习题
date: 2012-11-04T08:00:00+08:00
draft: false
toc:
comments: true
---


数据文件 datefile

	Steve Blenheim:238-923-7366:95 Latham Lane, Easton, PA 83755:11/12/56:20300
	Betty Boop:245-836-8357:635 Cutesy Lane, Hollywood, CA 91464:6/23/23:14500
	Igor Chevsky:385-375-8395:3567 Populus Place, Caldwell, NJ 23875:6/18/68:23400
	Norma Corder:397-857-2735:74 Pine Street, Dearborn, MI 23874:3/28/45:245700
	Jennifer Cowan:548-834-2348:583 Laurel Ave., Kingsville, TX 83745:10/1/35:58900
	Jon DeLoach:408-253-3122:123 Park St., San Jose, CA 04086:7/25/53:85100
	Karen Evich:284-758-2857:23 Edgecliff Place, Lincoln, NB 92743:7/25/53:85100
	Karen Evich:284-758-2867:23 Edgecliff Place, Lincoln, NB 92743:11/3/35:58200
	Karen Evich:284-758-2867:23 Edgecliff Place, Lincoln, NB 92743:11/3/35:58200
	Fred Fardbarkle:674-843-1385:20 Parak Lane, Duluth, MN 23850:4/12/23:780900
	Fred Fardbarkle:674-843-1385:20 Parak Lane, Duluth, MN 23850:4/12/23:780900
	Lori Gortz:327-832-5728:3465 Mirlo Street, Peabody, MA 34756:10/2/65:35200
	Paco Gutierrez:835-365-1284:454 Easy Street, Decatur, IL 75732:2/28/53:123500
	Ephram Hardy:293-259-5395:235 CarltonLane, Joliet, IL 73858:8/12/20:56700
	James Ikeda:834-938-8376:23445 Aster Ave., Allentown, NJ 83745:12/1/38:45000
	Barbara Kertz:385-573-8326:832 Ponce Drive, Gary, IN 83756:12/1/46:268500
	Lesley Kirstin:408-456-1234:4 Harvard Square, Boston, MA 02133:4/22/62:52600
	William Kopf:846-836-2837:6937 Ware Road, Milton, PA 93756:9/21/46:43500
	Sir Lancelot:837-835-8257:474 Camelot Boulevard, Bath, WY 28356:5/13/69:24500
	Jesse Neal:408-233-8971:45 Rose Terrace, San Francisco, CA 92303:2/3/36:25000
	Zippy Pinhead:834-823-8319:2356 Bizarro Ave., Farmount, IL 84357:1/1/67:89500
	Arthur Putie:923-835-8745:23 Wimp Lane, Kensington, DL 38758:8/31/69:126000
	Popeye Sailor:156-454-3322:945 Bluto Street, Anywhere, USA 29358:3/19/35:22350
	Jose Santiago:385-898-8357:38 Fife Way, Abilene, TX 39673:1/5/58:95600
	Tommy Savage:408-724-0140:1222 Oxbow Court, Sunnyvale, CA 94087:5/19/66:34200
	Yukio Takeshida:387-827-1095:13 Uno Lane, Ashville, NC 23556:7/1/29:57000
	Vinh Tranh:438-910-7449:8235 Maple Street, Wilmington, VM 29085:9/23/63:68900


* 把 Jon 的名字改为 Jonathan

		sed -e 's/^Jon /Jonathan /' datebook

* 删除前 3 行

		sed -e '1,3d' datebook

* 打印第 5~10 行

		sed -n -e '5,10p' datebook

* 删除含有 Lane 的所有行
	
		sed -e '/Lane/d' datebook

* 删除不含有 Lane 的所有行

		sed -e '/Lane/!d' datebook

* 打印所有生日在十一月或十二月的行

		sed -n -e '/:1[12]\//p' datebook

* 在以 James 开头的各行末尾加上 3 颗星号

		sed -e '/^James/s/$/***&/' datebook

* 将所有包含 Jose 的行都替换为 JOSE HAS RETIRED

		sed -e '/Jose/c\JOSE HAS RETIRED' datebook

* 把 Popeye 的生日改为 11/14/46 ，假定您不知道 Popeye 的生日，设法用正则表达式查找出来

		sed -n -e '/^Popeye /s/:[0-9]*\/[0-9]*\/[0-9]*/:11\/14\/46:/p' datebook

* 删除所有空行
	
		sed -e '/^$/d' datebook


* 写一个能完成下列任务的 sed 脚本

	a) 在第一行上方插入标题 PERSONNEL FILE

	b) 删除以 500 结尾的工资项

	c) 把名和姓的位置颠倒后，打印文件内容

	d) 在文件末尾加上 THE END

		1i\PERSONNEL FILE
		/500$/d
		s/\([a-zA-Z]*\) \([a-zA-Z]*\)/\2 \1/
		$a\THE END
