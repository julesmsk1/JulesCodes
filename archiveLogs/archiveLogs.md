#!/bin/ksh
echo ""
echo "#========================================================================================================================"
echo "#"
echo "#          FILE:  archiveLogs.ksh"
echo "#"
echo "#         USAGE:  archiveLogs -s -e [ -d | -v | -r | -i ]"
echo "#"
echo "#   DESCRIPTION:  This archive script has 3 basic functions:"
echo "#		1. Clean-up files older then 60 days or given retention period (-r)"
echo "#		2. Tar and Zip log files (*.log, *.out, specify by -e )"
echo "#		3. Make a daily, weekly and monthly incremental backups"
echo "#"
echo "#       OPTIONS:"
echo "#		  -s | --source  = Source location of files to backup (mandatory)"
echo "#		  -e | --extention = Filename extention String to search and zip (mandatory) max 5 extentions"
echo "#		  -d | --destination = Zip files destination (option)"
echo "#		  -r | --retention = retention period (option), default is 30 days"
echo "#		  -v | --version = current version (optiona)"
echo "#		  -h | --help = help script usage"
echo "#	    	  -i | --info = Script Info"
echo "#"
echo "#		  Default retention period is 30 days"
echo "#"
echo "#  REQUIREMENTS:  AIX 7.1 Environment"
echo "#          BUGS:  ---"
echo "#         NOTES:  ---"
echo "#        AUTHOR:  Jules Musoko | Jules@Musoko.com"
echo "#       COMPANY:  ATOS "
echo "#       VERSION:  1.2"
echo "#       CREATED:  24/08/2016"
echo "#      REVISION:  24/05/2017 - Version 1.2"
echo "#      RELEASED:  02/06/2017"
echo "#======================================================================================================================="
echo ""
exit 0
}

#How to use the script
usage() {
 echo ""
 echo "*********************************************************************"
 echo ""
 echo " Usage: $0 -s -e [-d | -v | -r | -i ]"
 echo ""
 echo " Example: $0 -s /directory/SourceDir/ -e *log.*"
 echo ""
 echo " Only user oracle can run the script !!!"
 echo ""
 echo " Options:"
 echo "          -s | --source  = Source location of files to backup (mandatory)"
 echo "          -e | --extention = Filename extention String to search and zip (mandatory) max 5 extentions"
 echo "          -d | --destination = Zip files destination (option)"
 echo "          -r | --retention = retention period (option), default is 30 days"
 echo "          -v | --version = current version (option)"
 echo "          -h | --help = help script usage"
 echo "          -i | --info = Script Info"
 echo ""
 echo "*********************************************************************"
 echo ""
exit 0
}


#Current version
version(){
echo ""
echo "VERSION:   1.2"
echo "CREATED:   24/08/2016"
echo "REVISION:  24/05/2017 - Version 1.2"
echo "RELEASED:  02/06/2017"
echo ""
exit 0
}

#Logging
logs() {
 dateAndTime=`date`
 msg="$1"
 echo "[$dateAndTime] $msg" >> $SCRIPT_LOG
}


#Archiving
archive() {
	sleep 1
	sleep 1
	sTIME=`date +"%Od%Om%Oy_%OH%M"`
	FILENAME="$1""_""$FNAME"
	file="$1"
	tar cpf - $file | gzip > $FILENAME &>/dev/null

	if [ "$?" = "0" ]; then
		logs "Archive successfull: $FILENAME"
		until [ -f $FILENAME ]
		do
			sleep 1
		done
		sleep 1
		rm $file &>/dev/null
		if [ "$?" != "0" ]; then
			logs "[ERROR] Unable to remove $extLog"
			logs "[ERROR] Script stopt with ERROR."
			exit
		fi
	else
		logs "[ERROR] occurs while archiving: $extLog"
		logs "[ERROR] unable to archive: $extLog"
		logs "[ERROR] Script stopt with ERROR."
		exit
	fi
}



#StarUp_checks
EXT1=""
EXT2=""
EXT3=""
EXT4=""
EXT5=""
if [ `whoami` != "oracle" ]; then usage; fi
if [ $# -eq 0 ]; then usage;fi
#if [ $# -lt 4 ]; then usage; fi
while (( "$#" )); do
	case "$1" in
		-h|--help) 
			usage
			;;
		-s|--source)
			SCRDIR="$2"
			shift 2
			;;
		-d|--destination)
			DESDIR="$2"
			shift 2
			;;
		-r|--retention)
			RTP="$2"
			shift 2
			;;
		-v|--version)
			version
			;;
		-i|--info)
			info
			;;
		-e|--extention)
			EXT1="$2"
			let shiftNr=2
			if [[ "$3" != -* ]] && [ "x$3" != "x" ]; then
				EXT2="$3"
				let shiftNr=$shiftNr+1
			fi
			if [[ "$4" != -* ]] && [ "$EXT2" != "" ] && [ "x$4" != "x" ]; then
                                EXT3="$4"
                                let shiftNr=$shiftNr+1
                        fi
			if [[ "$5" != -* ]] && [ "$EXT3" != "" ] && [ "x$5" != "x" ]; then
                                EXT4="$5"
                                let shiftNr=$shiftNr+1
                        fi
			if [[ "$6" != -* ]] && [ "$EXT4" != "" ] && [ "x$6" != "x" ]; then
                                EXT5="$6"
                                let shiftNr=$shiftNr+1
                        fi
			shift $shiftNr
			;;
		*)
			echo "Wrong argument: $1"
			usage
			;;
	esac
done

if [ "$SCRDIR" = "" ] || [ "$EXT1" = "" ]; then usage; fi
if [ "$DESDIR" = "" ]; then DESDIR="$SCRDIR""archives"; fi
if [ "$RTP" = "" ]; then RTP=30; fi


#2_Initialization
sTIME=`date +"%Od%Om%Oy_%OH%M%S"`
FNAME=$sTIME.tar.gz
sHOST=`hostname`
if [ ! -d $DESDIR ]; then mkdir $DESDIR;fi
SCRIPT_LOG="$DESDIR/archiveLogs.out"
if [ ! -f $SCRIPT_LOG ]; then touch $SCRIPT_LOG; fi
echo ".................................. \r" >> $SCRIPT_LOG
logs "Archiving Script started from $sHOST"
logs "Date(DDMMYY) & time(HHMMSS) = $sTIME"
logs "Archive directory: $SCRDIR"
logs "Archive destination: $DESDIR"
logs "Retention period: $RTP"
logs "Extention1 = $EXT1"
if [ "$EXT2" != "" ]; then logs "Extention2 = $EXT2";fi
if [ "$EXT3" != "" ]; then logs "Extention3 = $EXT3";fi
if [ "$EXT4" != "" ]; then logs "Extention4 = $EXT4";fi
if [ "$EXT5" != "" ]; then logs "Extention5 = $EXT5";fi
#exit 0


#Remove old archive files: older then 30 or equal do retention periode
logs "Removing archive file(s) older then $RTP days from $DESDIR"
let nrRmFiles=0
for rmfile in `find $DESDIR -type f -name "*.gz" -mtime +$RTP`
do
	rm $rmfile
	let nrRmFiles=$nrRmFiles+1
	logs "Old Archive file deleted: $rmfile"
done
logs "$nrRmFiles total number of file(s) deleted with $RTP days retention periode"


#4_Tar and zip files
logs "Start archiving files with extention: $EXT1"
for extfile1 in `find $SCRDIR -type f -name "$EXT1" ! -name "*.gz"`
do
	archive $extfile1
done
if [ "$EXT2" != "" ]; then
	logs "Start archiving files with extention: $EXT2"
	for extfile2 in `find $SCRDIR -type f -name "$EXT2" ! -name "*.gz"`
	do
		archive $extfile2
	done
fi
if [ "$EXT3" != "" ]; then
	logs "Start archiving files with extention: $EXT3"
	for extfile3 in `find $SCRDIR -type f -name "$EXT3" ! -name "*.gz"`
        do
		archive $extfile3
        done
fi
if [ "$EXT4" != "" ]; then
	logs "Start archiving files with extention: $EXT4"
	for extfile4 in `find $SCRDIR -type f -name "$EXT4" ! -name "*.gz"`
	do
		archive $extfile4
	done
fi
if [ "$EXT5" != "" ]; then
	logs "Start archiving files with extention: $EXT5"
	for extfile5 in `find $SCRDIR -type f -name "$EXT4" ! -name "*.gz"`
	do
		archive $extfile5
	done
fi

#4_Finalize
if [ -e $SCRDIR/*.gz ]; then
	mv $(find $SCRDIR -type f -name "*.gz") $DESDIR
	logs "Moved tar|zip files to archives directory: $DESDIR"
fi
fTIME=`date +"%Od%Om%Oy_%OH%M%S"`
#SCRIPT_LOG2=$SCRIPT_LOG"_"$fTIME
logs "Script execution ended at: $fTIME"
#mv $SCRIPT_LOG $SCRIPT_LOG2
