---
layout: post
category: Search
title: 《Nutch2.3 bin/crawl、bin/nutch 脚本》笔记
tagline: by Vv
tags: Nutch
---

# 环境 #
***
<br/>

	Nutch版本：Nutch 2.3

# 目录 #
***
<br/>

	1.bin/crawl脚本
	2.bin/nutch脚本

# 内容 #
***
<br/>

## 1.bin/crawl脚本 ##

    #!/bin/bash
    # The Crawl command script : crawl <seedDir> <crawlId> <solrURL> <numberOfRounds>
    #
    # 下面这一段主要是判断bin/crawl命令的参数
	#
    # UNLIKE THE NUTCH ALL-IN-ONE-CRAWL COMMAND THIS SCRIPT DOES THE LINK INVERSION AND 
    # INDEXING FOR EACH BATCH
	
	SEEDDIR="$1"
	CRAWL_ID="$2"
	if [ "$#" -eq 3 ]; then
    	LIMIT="$3"
	elif [ "$#" -eq 4 ]; then
    	 SOLRURL="$3"
     	LIMIT="$4"
	else
    	echo "Unknown # of arguments $#"
    	echo "Usage: crawl <seedDir> <crawlID> [<solrUrl>] <numberOfRounds>"
    	exit -1;
	fi

	if [ "$SEEDDIR" = "" ]; then
    	echo "Missing seedDir : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>"
    	exit -1;
	fi

	if [ "$CRAWL_ID" = "" ]; then
    	echo "Missing crawlID : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>"
    	exit -1;
	fi

	if [ "$SOLRURL" = "" ]; then
    	echo "No SOLRURL specified. Skipping indexing."
	fi

	if [ "$LIMIT" = "" ]; then
    	echo "Missing numberOfRounds : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>"
		exit -1;
	fi

	#下面的这段是可以根据实际环境需要进行配置的
	#
    #############################################
    # MODIFY THE PARAMETERS BELOW TO YOUR NEEDS #
    #############################################

    # set the number of slaves nodes 设置Hadoop集群环境中Slave节点的数量
    numSlaves=1
    
    # and the total number of available tasks 分布计算的任务数
    # sets Hadoop parameter "mapred.reduce.tasks"
    numTasks=`expr $numSlaves \* 2`
    
    # number of urls to fetch in one iteration
    # 250K per task?
    sizeFetchlist=`expr $numSlaves \* 50000`
    
    # time limit for feching
    timeLimitFetch=180
    
    # Adds <days> to the current time to facilitate 
    # crawling urls already fetched sooner then 按天设置爬取间隔 
    # db.default.fetch.interval.
    addDays=0
    #############################################
    
    bin="`dirname "$0"`"
    bin="`cd "$bin"; pwd`"

	# 根据是否存在job文件来判断是分布式或者本地运行
	# determines whether mode based on presence of job file
	mode=local
	if [ -f "${bin}"/../*nutch*.job ]; then
    	mode=distributed
	fi

	# Hadoop的一些参数
    # note that some of the options listed here could be set in the 
    # corresponding hadoop site xml param file 
    commonOptions="-D mapred.reduce.tasks=$numTasks -D mapred.child.java.opts=-Xmx1000m -D mapred.reduce.tasks.speculative.execution=false -D mapred.map.tasks.speculative.execution=false -D mapred.compress.map.output=true"

	# 检查是否设置Hadoop环境变量
 	# check that hadoop can be found on the path 
	if [ $mode = "distributed" ]; then
	 if [ $(which hadoop | wc -l ) -eq 0 ]; then
    	echo "Can't find Hadoop executable. Add HADOOP_HOME/bin to the path or run in local mode."
    	exit -1;
	 fi
	fi


	function __bin_nutch {
    	# run $bin/nutch, exit if exit value indicates error

   		echo "$bin/nutch $@" ;# echo command and arguments
    	"$bin/nutch" "$@"

    	RETCODE=$?
    	if [ $RETCODE -ne 0 ]
    	then
        	echo "Error running:"
        	echo "  $bin/nutch $@"
        	echo "Failed with exit value $RETCODE."
        	exit $RETCODE
    	fi
	}



	# initial injection
	echo "Injecting seed URLs"
	__bin_nutch inject "$SEEDDIR" -crawlId "$CRAWL_ID"

	# 根据LIMIT参数设置循环次数
	# main loop : rounds of generate - fetch - parse - update
	for ((a=1; a <= LIMIT ; a++))
	do
	  if [ -e ".STOP" ]
	  then
	   echo "STOP file found - escaping loop"
	   break
	  fi

	  echo `date` ": Iteration $a of $LIMIT"

	  echo "Generating batchId"
	  batchId=`date +%s`-$RANDOM

		# 执行Generat操作
	  echo "Generating a new fetchlist"
	  generate_args=($commonOptions -topN $sizeFetchlist -noNorm -noFilter -adddays $addDays -crawlId "$CRAWL_ID" -batchId $batchId)
	  echo "$bin/nutch generate ${generate_args[@]}"
	  $bin/nutch generate "${generate_args[@]}"
	  RETCODE=$?
	  if [ $RETCODE -eq 0 ]; then
	      : # ok: no error
	  elif [ $RETCODE -eq 1 ]; then
	    echo "Generate returned 1 (no new segments created)"
	    echo "Escaping loop: no more URLs to fetch now"
	    break
	  else
	    echo "Error running:"
	    echo "  $bin/nutch generate ${generate_args[@]}"
	    echo "Failed with exit value $RETCODE."
	    exit $RETCODE
	  fi

		# 执行Fetch操作
	  echo "Fetching : "
	  __bin_nutch fetch $commonOptions -D fetcher.timelimit.mins=$timeLimitFetch $batchId -crawlId "$CRAWL_ID" -threads 50

		# 执行Parse操作
	  # parsing the batch
	  echo "Parsing : "
	  # enable the skipping of records for the parsing so that a dodgy document 
	  # so that it does not fail the full task
      skipRecordsOptions="-D mapred.skip.attempts.to.start.skipping=2 -D mapred.skip.map.max.skip.records=1"
      __bin_nutch parse $commonOptions $skipRecordsOptions $batchId -crawlId "$CRAWL_ID"

		# 执行updatedb操作
      # updatedb with this batch
      echo "CrawlDB update for $CRAWL_ID"
      __bin_nutch updatedb $commonOptions $batchId -crawlId "$CRAWL_ID"
	
		
	  if [ -n "$SOLRURL" ]; then
		# 执行index操作
	    echo "Indexing $CRAWL_ID on SOLR index -> $SOLRURL"
	    __bin_nutch index $commonOptions -D solr.server.url=$SOLRURL -all -crawlId "$CRAWL_ID"
		
		# 执行solrdedup操作
	    echo "SOLR dedup -> $SOLRURL"
	    __bin_nutch solrdedup $commonOptions $SOLRURL
	  else
	      echo "Skipping indexing tasks: no SOLR url provided."
	  fi

	done

	exit 0


	
## 2.bin/crawl脚本 ##

#!/bin/bash

	#
	# The Nutch command script
	#
	# Environment Variables 环境变量
	#
	#   NUTCH_JAVA_HOME The java implementation to use.  Overrides JAVA_HOME.
	#
	#   NUTCH_HEAPSIZE  The maximum amount of heap to use, in MB. 
	#                   Default is 1000.
	#
	#   NUTCH_OPTS      Extra Java runtime options.
	#                   Multiple options must be separated by white space.
	#
	#   NUTCH_LOG_DIR   Log directory (default: $NUTCH_HOME/logs)
	#
	#   NUTCH_LOGFILE   Log file (default: hadoop.log)
	#
	#   NUTCH_CONF_DIR  Path(s) to configuration files (default: $NUTCH_HOME/conf).
	#                   Multiple paths must be separated by a colon ':'.
	# 
	# cygwin是windows下的Linux运行环境
	cygwin=false
	case "`uname`" in
	CYGWIN*) cygwin=true;;
	esac

	# resolve links - $0 may be a softlink
	THIS="$0"
	while [ -h "$THIS" ]; do
	  ls=`ls -ld "$THIS"`
	  link=`expr "$ls" : '.*-> \(.*\)$'`
	  if expr "$link" : '.*/.*' > /dev/null; then
	    THIS="$link"
	  else
	    THIS=`dirname "$THIS"`/"$link"
	  fi
	done
	
	# nutch命令参数信息
	# if no args specified, show usage
	if [ $# = 0 ]; then
	  echo "Usage: nutch COMMAND"
	  echo "where COMMAND is one of:"
	  echo " inject		inject new urls into the database"
	  echo " hostinject     creates or updates an existing host table from a text file"
	  echo " generate 	generate new batches to fetch from crawl db"
	  echo " fetch 		fetch URLs marked during generate"
	  echo " parse 		parse URLs marked during fetch"
	  echo " updatedb 	update web table after parsing"
	  echo " updatehostdb   update host table after parsing"
	  echo " readdb 	read/dump records from page database"
	  echo " readhostdb     display entries from the hostDB"
	  echo " index          run the plugin-based indexer on parsed batches"
	  echo " elasticindex   run the elasticsearch indexer - DEPRECATED use the index command instead"
	  echo " solrindex 	run the solr indexer on parsed batches - DEPRECATED use the index command instead"
	  echo " solrdedup 	remove duplicates from solr"
	  echo " solrclean      remove HTTP 301 and 404 documents from solr - DEPRECATED use the clean command instead"
	  echo " clean          remove HTTP 301 and 404 documents and duplicates from indexing backends configured via plugins"
	  echo " parsechecker   check the parser for a given url"
	  echo " indexchecker   check the indexing filters for a given url"
	  echo " plugin 	load a plugin and run one of its classes main()"
	  echo " nutchserver    run a (local) Nutch server on a user defined port"
	  echo " webapp         run a local Nutch web application"
	  echo " junit         	runs the given JUnit test"
	  echo " or"
	  echo " CLASSNAME 	run the class named CLASSNAME"
	  echo "Most commands print help when invoked w/o parameters."
	  exit 1
	fi

	# get arguments
	COMMAND=$1
	shift
	
	# 根据当前目录设置NUTCH_HOME
	# some directories
	THIS_DIR="`dirname "$THIS"`"
	NUTCH_HOME="`cd "$THIS_DIR/.." ; pwd`"

	# some Java parameters
	if [ "$NUTCH_JAVA_HOME" != "" ]; then
	  #echo "run java in $NUTCH_JAVA_HOME"
	  JAVA_HOME="$NUTCH_JAVA_HOME"
	fi
  
	if [ "$JAVA_HOME" = "" ]; then
	  echo "Error: JAVA_HOME is not set."
	  exit 1
	fi


	# NUTCH_JOB 
	if [ -f "${NUTCH_HOME}"/*nutch*.job ]; then
	  local=false
	  for f in "$NUTCH_HOME"/*nutch*.job; do
	    NUTCH_JOB="$f";
	  done
	  # cygwin path translation
	  if $cygwin; then
	    NUTCH_JOB="`cygpath -p -w "$NUTCH_JOB"`"
	  fi
	else
	  local=true
	fi

	JAVA="$JAVA_HOME/bin/java"
	JAVA_HEAP_MAX=-Xmx1000m 

	# check envvars which might override default args
	if [ "$NUTCH_HEAPSIZE" != "" ]; then
	  #echo "run with heapsize $NUTCH_HEAPSIZE"
	  JAVA_HEAP_MAX="-Xmx""$NUTCH_HEAPSIZE""m"
	  #echo $JAVA_HEAP_MAX
	fi
	
	# 把NUTCH_HOME添加到CLASSPATH
	# CLASSPATH initially contains $NUTCH_CONF_DIR, or defaults to $NUTCH_HOME/conf
	CLASSPATH="${NUTCH_CONF_DIR:=$NUTCH_HOME/conf}"
	CLASSPATH="${CLASSPATH}:$JAVA_HOME/lib/tools.jar"

	# so that filenames w/ spaces are handled correctly in loops below
	IFS=

	# add libs to CLASSPATH
	if $local; then
	  for f in "$NUTCH_HOME"/lib/*.jar; do
	   CLASSPATH="${CLASSPATH}:$f";
	  done
	  # local runtime
	  # add plugins to classpath
	  if [ -d "$NUTCH_HOME/plugins" ]; then
     CLASSPATH="${NUTCH_HOME}:${CLASSPATH}"
	  fi
	fi

	# cygwin path translation
	if $cygwin; then
	  CLASSPATH="`cygpath -p -w "$CLASSPATH"`"
	fi

	# setup 'java.library.path' for native-hadoop code if necessary
	# used only in local mode 
	JAVA_LIBRARY_PATH=''
	if [ -d "${NUTCH_HOME}/lib/native" ]; then

	  JAVA_PLATFORM=`"${JAVA}" -classpath "$CLASSPATH" org.apache.hadoop.util.PlatformName | sed -e 's/ /_/g'`

	  if [ -d "${NUTCH_HOME}/lib/native" ]; then
	    if [ "x$JAVA_LIBRARY_PATH" != "x" ]; then
	      JAVA_LIBRARY_PATH="${JAVA_LIBRARY_PATH}:${NUTCH_HOME}/lib/native/${JAVA_PLATFORM}"
	    else
	      JAVA_LIBRARY_PATH="${NUTCH_HOME}/lib/native/${JAVA_PLATFORM}"
	    fi
	  fi
	fi

	if [ $cygwin = true -a "X${JAVA_LIBRARY_PATH}" != "X" ]; then
	  JAVA_LIBRARY_PATH="`cygpath -p -w "$JAVA_LIBRARY_PATH"`"
	fi

	# restore ordinary behaviour
	unset IFS

	# default log directory & file
	if [ "$NUTCH_LOG_DIR" = "" ]; then
	  NUTCH_LOG_DIR="$NUTCH_HOME/logs"
	fi
	if [ "$NUTCH_LOGFILE" = "" ]; then
	  NUTCH_LOGFILE='hadoop.log'
	fi

	#Fix log path under cygwin
	if $cygwin; then
	  NUTCH_LOG_DIR="`cygpath -p -w "$NUTCH_LOG_DIR"`"
	fi

	NUTCH_OPTS=($NUTCH_OPTS -Dhadoop.log.dir="$NUTCH_LOG_DIR")
	NUTCH_OPTS=("${NUTCH_OPTS[@]}" -Dhadoop.log.file="$NUTCH_LOGFILE")

	if [ "x$JAVA_LIBRARY_PATH" != "x" ]; then
	  NUTCH_OPTS=("${NUTCH_OPTS[@]}" -Djava.library.path="$JAVA_LIBRARY_PATH")
	fi

	# 根据不同的参数调用不同的类
	# figure out which class to run
    if [ "$COMMAND" = "crawl" ] ; then
      echo "Command $COMMAND is deprecated, please use bin/crawl instead"
      exit -1
    elif [ "$COMMAND" = "inject" ] ; then
    CLASS=org.apache.nutch.crawl.InjectorJob
    elif [ "$COMMAND" = "hostinject" ] ; then
    CLASS=org.apache.nutch.host.HostInjectorJob
    elif [ "$COMMAND" = "generate" ] ; then
    CLASS=org.apache.nutch.crawl.GeneratorJob
    elif [ "$COMMAND" = "fetch" ] ; then
    CLASS=org.apache.nutch.fetcher.FetcherJob
    elif [ "$COMMAND" = "parse" ] ; then
    CLASS=org.apache.nutch.parse.ParserJob
    elif [ "$COMMAND" = "updatedb" ] ; then
    CLASS=org.apache.nutch.crawl.DbUpdaterJob
    elif [ "$COMMAND" = "updatehostdb" ] ; then
    CLASS=org.apache.nutch.host.HostDbUpdateJob
    elif [ "$COMMAND" = "readdb" ] ; then
    CLASS=org.apache.nutch.crawl.WebTableReader
    elif [ "$COMMAND" = "readhostdb" ] ; then
    CLASS=org.apache.nutch.host.HostDbReader
    elif [ "$COMMAND" = "elasticindex" ] ; then
    CLASS=org.apache.nutch.indexer.elastic.ElasticIndexerJob
    elif [ "$COMMAND" = "solrindex" ] ; then
    CLASS="org.apache.nutch.indexer.IndexingJob -D solr.server.url=$1"
    shift
    elif [ "$COMMAND" = "index" ] ; then
    CLASS=org.apache.nutch.indexer.IndexingJob
    elif [ "$COMMAND" = "solrdedup" ] ; then
    CLASS=org.apache.nutch.indexer.solr.SolrDeleteDuplicates
    elif [ "$COMMAND" = "solrclean" ] ; then
      CLASS="org.apache.nutch.indexer.CleaningJob -D solr.server.url=$2 $1"
      shift; shift
    elif [ "$COMMAND" = "clean" ] ; then
      CLASS=org.apache.nutch.indexer.CleaningJob
    elif [ "$COMMAND" = "parsechecker" ] ; then
      CLASS=org.apache.nutch.parse.ParserChecker
    elif [ "$COMMAND" = "indexchecker" ] ; then
      CLASS=org.apache.nutch.indexer.IndexingFiltersChecker
    elif [ "$COMMAND" = "plugin" ] ; then
    CLASS=org.apache.nutch.plugin.PluginRepository
    elif [ "$COMMAND" = "webapp" ] ; then
    CLASS=org.apache.nutch.webui.NutchUiServer
    elif [ "$COMMAND" = "nutchserver" ] ; then
    CLASS=org.apache.nutch.api.NutchServer
    elif [ "$COMMAND" = "junit" ] ; then
      CLASSPATH="$CLASSPATH:$NUTCH_HOME/test/classes/"
      CLASS=org.junit.runner.JUnitCore
    else
    CLASS=$COMMAND
    fi

	# 判断是否分布式运行	
    if $local; then
     # fix for the external Xerces lib issue with SAXParserFactory
     NUTCH_OPTS=(-Djavax.xml.parsers.DocumentBuilderFactory=com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl "${NUTCH_OPTS[@]}")
     EXEC_CALL=("$JAVA" $JAVA_HEAP_MAX "${NUTCH_OPTS[@]}" -classpath "$CLASSPATH")
    else
     # check that hadoop can be found on the path
     if [ $(which hadoop | wc -l ) -eq 0 ]; then
	    echo "Can't find Hadoop executable. Add HADOOP_HOME/bin to the path or run in local mode."
	    exit -1;
  	 fi
     # distributed mode
     EXEC_CALL=(hadoop jar "$NUTCH_JOB")
    fi
    
	# 根据前面设置好的参数执行命令
    # run it
    exec "${EXEC_CALL[@]}" $CLASS "$@"
    
    

