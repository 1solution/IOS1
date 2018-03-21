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
function call_weditor(){
	now="$(date +'%Y-%m-%d')"
	if grep -q "$wfile" $WEDI_RC; then
	#####->SED EDIT LINE
	echo ""
	else
	printf "%s\t1\t%s\n" "$wfile" "$now" >> $WEDI_RC
	fi
	#"${EDITOR:-${VISUAL:-vi}}" "$wfile"
	echo "Opening file $wfile in wrapper"
	return 0
}

if [[ -z $WEDI_RC ]]; then #is not set, then error
echo "WEDI_RC not set."
exit 1
elif [[ ! -f $WEDI_RC ]]; then #test if file not exists (but is set), then create path+file 
mkdir -p "$(dirname "$WEDI_RC")" && touch "$WEDI_RC"
fi

if [[ -f $1 ]]; then #if is file
	wfile="$(realpath "$1")"
	if [[ -f $wfile ]]; then
	echo "proslo validaci na soubor"
	call_weditor "$wfile"
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
		def_dir=$(realpath "$dr")
		else
		def_dir=$PWD
		fi
	wfile="$(awk -v dir="$def_dir" -v now="1955-05-05" 'BEGIN{-F "\t"} $1 ~ dir {if($3>now){now=$3;rcd=$1}} END{print rcd}' "$WEDI_RC")"
	call_weditor "$wfile"
fi

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
	awk -v dir="$def_dir" 'BEGIN{-F "\t"} $1 ~ dir {print $1}' "$WEDI_RC"
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
	awk -v dir="$def_dir" -v time="$time" 'BEGIN{-F "\t"} $1 ~ dir {if(time>=$3) {print $1}}' "$WEDI_RC"
	exit 0
	;;

	-a)
	dr=$3
	if [[ -d $dr ]] && [[ $(dir_exists "$dr") -eq 0 ]]; then
	def_dir=$(realpath "$dr")
	else
	def_dir=$PWD
	fi
	time=$2
	awk -v dir="$def_dir" -v time="$time" 'BEGIN{-F "\t"} $1 ~ dir {if(time<=$3) {print $1}}' "$WEDI_RC"	
	exit 0
	;;

esac
