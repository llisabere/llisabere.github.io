nmols=$1
firstgro=$2
itrial=1
x=6 #6 for normal 100k runs
y=6
mass=$(echo "scale=4; 1250.39*1.66" | bc)
#rho=360
z=$3 #8 for 50 pep, 16 for 100 pep try to maintain rho 320-360
rho=$(echo "scale=4; $mass*$nmols/$x/$y/$z" | bc)
echo Density expected $rho

filenum=`ls initial*gro | wc -l`
ifile=0

outfile=init.${nmols}peptides_box$x$y$z.gro

#gmx editconf -f $firstgro -o $outfile -c -box $x $y $z 
#cp $firstgro init.${nmols}peptides.gro

z=$(($z-10))
echo Part one z equal $z

gmx editconf -f $firstgro -o $outfile -c -box $x $y $z

rm -f success.log
echo 'success' > success.log
for gro in `ls -t initial*.gro`
do
        rm -f out.log
        imols=`awk 'NR==2{print $1/176 ;exit}' $outfile ` # init.${nmols}peptides.gro`
        #imols=$(echo "scale=4; $imols" | bc)
        echo -ne "$ifile / $filenum .gro avail and $imols / $nmols nmols put ; trial $itrial"\\r
        ((ifile++))
        #if [ $itrial -ge $nmols ]
        if [ $imols -ge $nmols ]
        then
                break
        fi
        rm -f \#*\#
        if [ $gro == $firstgro ]
        then
                continue
        else
                #echo -ne "$itrial $gro "\\r
                gmx insert-molecules -f $outfile -ci $gro -nmol 1 -o $outfile -box $x $y $z -try 200 &> out.log
                grep 'now' out.log >> success.log
                itrial="$(wc -l < success.log)"
        fi
done
echo "$ifile / $filenum and $imols / $nmols"
rhofin=$(echo "scale=4; $mass*$imols/$x/$y/$z" | bc)
echo Final density $rhofin
rm -f out.log
rm -f success.log
echo "Done creating $outfile"

z=$(($z+5))
echo Part two z equal $z

gmx editconf -f $outfile -o $outfile -c -box $x $y $z
gmx editconf -f $outfile -o $outfile -c -box $x $y $z

rm -f success.log
echo 'success' > success.log
for gro in `ls -t initial*.gro`
do
        rm -f out.log
        imols=`awk 'NR==2{print $1/176 ;exit}' $outfile ` # init.${nmols}peptides.gro`
        #imols=$(echo "scale=4; $imols" | bc)
        echo -ne "$ifile / $filenum .gro avail and $imols / $nmols nmols put ; trial $itrial"\\r
        ((ifile++))
        #if [ $itrial -ge $nmols ]
        if [ $imols -ge $nmols ]
        then
                break
        fi
        rm -f \#*\#
        if [ $gro == $firstgro ]
        then
                continue
        else
                #echo -ne "$itrial $gro "\\r
                gmx insert-molecules -f $outfile -ci $gro -nmol 1 -o $outfile -box $x $y $z -try 200 &> out.log
                grep 'now' out.log >> success.log
                itrial="$(wc -l < success.log)"
        fi
done
echo "$ifile / $filenum and $imols / $nmols"
rhofin=$(echo "scale=4; $mass*$imols/$x/$y/$z" | bc)
echo Final density $rhofin
rm -f out.log
rm -f success.log
echo "Done creating $outfile"

z=$(($z+5))
echo "Final part (lets hope!) z equal $z"

gmx editconf -f $outfile -o $outfile -c -box $x $y $z

rm -f success.log
echo 'success' > success.log
for gro in `ls -t initial*.gro`
do
        rm -f out.log
        imols=`awk 'NR==2{print $1/176 ;exit}' $outfile ` # init.${nmols}peptides.gro`
        #imols=$(echo "scale=4; $imols" | bc)
        echo -ne "$ifile / $filenum .gro avail and $imols / $nmols nmols put ; trial $itrial"\\r
        ((ifile++))
        #if [ $itrial -ge $nmols ]
        if [ $imols -ge $nmols ]
        then
                break
        fi
        rm -f \#*\#
        if [ $gro == $firstgro ]
        then
                continue
        else
                #echo -ne "$itrial $gro "\\r
                gmx insert-molecules -f $outfile -ci $gro -nmol 1 -o $outfile -box $x $y $z -try 200 &> out.log
                grep 'now' out.log >> success.log
                itrial="$(wc -l < success.log)"
        fi
done
echo "$ifile / $filenum and $imols / $nmols"
rhofin=$(echo "scale=4; $mass*$imols/$x/$y/$z" | bc)
echo Final density $rhofin
rm -f out.log
rm -f success.log
echo "Done creating $outfile"

echo "Final final part lol z equal $z"

gmx editconf -f $outfile -o $outfile -c -box $x $y $z

rm -f success.log
echo 'success' > success.log
for gro in `ls -t initial*.gro`
do
        rm -f out.log
        imols=`awk 'NR==2{print $1/176 ;exit}' $outfile ` # init.${nmols}peptides.gro`
        #imols=$(echo "scale=4; $imols" | bc)
        echo -ne "$ifile / $filenum .gro avail and $imols / $nmols nmols put ; trial $itrial"\\r
        ((ifile++))
        #if [ $itrial -ge $nmols ]
        if [ $imols -ge $nmols ]
        then
                break
        fi
        rm -f \#*\#
        if [ $gro == $firstgro ]
        then
                continue
        else
                #echo -ne "$itrial $gro "\\r
                gmx insert-molecules -f $outfile -ci $gro -nmol 1 -o $outfile -box $x $y $z -try 200 &> out.log
                grep 'now' out.log >> success.log
                itrial="$(wc -l < success.log)"
        fi
done
echo "$ifile / $filenum and $imols / $nmols"
rhofin=$(echo "scale=4; $mass*$imols/$x/$y/$z" | bc)
echo Final density $rhofin
rm -f out.log
rm -f success.log
echo "Done creating $outfile"

