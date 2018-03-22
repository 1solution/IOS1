#!/usr/bin/env bash
POSIXLY_CORRECT=yes
#command -v realpath >/dev/null 2>&1 || (echo >&2 "Realpath utility not available.";exit 1)

##rozjet: if file is in WEDI_RC but not exists -> delete. chyby z editoru predavat. pocitat u most i s casem r! cleaner pri kazdem spusteni se zavola na zacatku a vycisti WEDI_RC od sracek

#check if dir is set as argument and if exists
function dir_exists(){
	if [[ -n $dr ]] && [[ -d $dr ]]; then
	return 0
	else
	return 1
	fi
}
function call_weditor(){
	now="$(date +'%Y-%m-%d')"
	if grep -q "$wfile" "$WEDI_RC"; then
	wcount="$(awk -v dir="$def_dir" 'BEGIN{-F "\t"} $1 ~ dir {rcd=$2} END{print rcd}' "$WEDI_RC")" # find count with awk
	let "wcount++"
	sed -i "s/.*$wfile.*/$wfile\t$wcount\t$now/"
	else
	printf "%s\t1\t%s\n" "$wfile" "$now" >> $WEDI_RC
	fi
	"${EDITOR:-${VISUAL:-vi}}" "$wfile"
	return 0
}
#function cleaner(){
#	
#}

if [[ -z $WEDI_RC ]]; then #is not set, then error
echo "WEDI_RC not set."
exit 1
elif [[ ! -f $WEDI_RC ]]; then #test if file not exists (but is set), then create path+file 
mkdir -p "$(dirname "$WEDI_RC")" && touch "$WEDI_RC"
fi

if [[ $1 == -b || $1 == -l || $1 == -a || $1 == -m ]]; then

case "$1" in
	-m)
	dr=$2
	if [[ -d $dr ]] && [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	wfile="$(awk -v max=0 -v dir="$def_dir" 'BEGIN{-F "\t"} $1 ~ dir {if($2>max){max=$2; rcd=$1}} END{print rcd}' "$WEDI_RC")"
	call_weditor "$wfile"
	;;

	-l)
	dr=$2
	if [[ -d $dr ]] && [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	awk -v dir="$def_dir" 'BEGIN{-F" +|/"} $1 ~ dir {n=split($1,a,"/"); print a[n]}' "$WEDI_RC"
	exit 0
	;;

	-b)
	dr=$3
	if [[ -d $dr ]] && [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	time=$2
	if [[ $time =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]; then
	awk -v dir="$def_dir" -v time="$time" 'BEGIN{-F "\t"} $1 ~ dir {if(time>=$3) {n=split($1,a,"/"); print a[n]}}' "$WEDI_RC"
	exit 0
	else
	echo "Incorrect date format."
	exit 1
	fi
	;;

	-a)
	dr=$3
	if [[ -d $dr ]] && [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	time=$2
	if [[ $time =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]; then
	awk -v dir="$def_dir" -v time="$time" 'BEGIN{-F "\t"} $1 ~ dir {if(time<=$3) {n=split($1,a,"/"); print a[n]}}' "$WEDI_RC"
	exit 0
	else
	echo "Incorrect date format."
	exit 1
	fi
	;;

esac


elif [[ -f $(realpath $1) ]]; then #if is file
	wfile="$(realpath "$1")"
	call_weditor "$wfile"	

elif [[ -d $1 ]] || [[ -z $1 ]]; then #if is DIR	
		if [[ -d $1 ]]; then
		dr=$1
		else
		dr=$PWD
		fi	
		if [[ $(dir_exists "$dr") -eq 0 ]]; then
		def_dir=$(realpath "$dr")
		else
		def_dir=$PWD
		fi
	wfile="$(awk -v dir="$def_dir" -v now="1955-05-05" 'BEGIN{-F "\t"} $1 ~ dir {if($3>now){now=$3;rcd=$1}} END{print rcd}' "$WEDI_RC")"
	call_weditor "$wfile"

else
	echo "Not a file."
	exit 1
fi
