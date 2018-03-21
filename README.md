#!/usr/bin/env bash
POSIXLY_CORRECT=yes

#check if dir is set as argument and if exists
function dir_exists(){
	if [[ -n $dr ]] && [[ -d $dr ]]; then
	return 0
	else
	return 1
	fi
}
#check if file exists in realpath
function check_file(){
	if [[ -f $(realpath "$wfile") ]]; then
	return 0
	else
	return 1
	fi
}
function call_weditor(){
	"${EDITOR:-${VISUAL:-vi}}" "$wfile"
	#now="$(date +'%Y/%m/%d')"
	#res="$(awk 'BEGIN -F "\t" -v file="$wfile" -v w=$WEDI_RC /$file/ {if($1 !~ "$file"){print "1"}; else {print "0"}}' "$w")"
	if [[ $res -eq 0 ]]; then #is in the file
	#awk 'BEGIN -F "\t" -v new=$now -v file="$wfile" -v w=$WEDI_RC /$file/ {$2++; $3=$now}' "$w"
	echo "call weditor awk wall"
	else
	printf "%s\t1\t%s\n" "$wfile" "$now" >> $WEDI_RC
	fi	
	return 0 ##### ?????
}

if [[ -z $WEDI_RC ]]; then #is not set, then error
echo "WEDI_RC not set."
exit 1
elif [[ ! -f $WEDI_RC ]]; then #test if file not exists (but is set), then create path+file 
mkdir -p "$(dirname "$WEDI_RC")" && touch "$WEDI_RC"
fi

if [[ -f $1 ]]; then #if is file
	wfile=$1
	if [[ $(check_file "$wfile") -eq 0 ]]; then
	call_weditor "$(realpath "$wfile")"
	else
	echo "File not exists."
	exit 1
	fi	

elif [[ -d $1 ]] || [[ -z $1 ]]; then #if is DIR	
		if [[ -d $1 ]]; then
		dr=$1
		else
		dr=$PWD
		fi	
		if [[ $(dir_exists "$dr") -eq 0 ]]; then
		echo "navrat z fce dir exists"
		def_dir=$(realpath "$dr")
		else
		def_dir=$PWD
		fi
	echo "IS dir or empty dir awk wall"
	wfile="$(awk -v dir="$def_dir" -v now="1985-01-01" 'BEGIN{-F "\t"} $1 ~ dir {if($3>now) now=$3;rcd=$1} END{print rcd}' "$WEDI_RC")" ## IS O.K.
	echo "validation flag"
	echo "$wfile"
	#call_weditor "$wfile"
fi

case "$1" in

	-m)
	dr=$2
	if [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	#wfile="$(awk 'BEGIN -F "\t" -v max=0 -v dir="$def_dir" -v w="$WEDI_RC" /$def_dir/ {if($2>max){final=$1; max=$2}} END{print final}' "$w")"
	#call_weditor "$wfile"
	;;

	-l)
	dr=$2
	if [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	#awk 'BEGIN -F "\t" -v dir="$def_dir" -v w="$WEDI_RC" /$def_dir/ {print $0}' "$w"
	return 0
	;;

	-b)
	dr=$3
	if [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	tim=$2
	#awk 'BEGIN -F "\t" -v t="$tim" -v dir="$def_dir" -v w=$WEDI_RC /$def_dir/ {if(t>=$3) {print}}' "$w"
	return 0	
	;;

	-a)
	dr=$3
	if [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	tim=$2
	#awk 'BEGIN -F "\t" -v t="$tim" -v dir="$def_dir" -v w=$WEDI_RC /$def_dir/ {if(t<=$3) {print}}' "$w"	
	return 0
	;;

esac
