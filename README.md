# ssr_finder

Find simple repeat sequences in the assembly genome and design proper primers for simple repeat sequences in 10 steps
ssr_finder

Find simple repeat sequences in the assembly genome and design proper primers for simple repeat sequences

####################To get ssr from the genome################### 
1ssr_finder: to get ssr,find the tandem repeat from the target sequences ,you can define ssr repeat numbers

and the flank length of ssr yourself or you can use default (motif two >= six times, motif three, motif four, motif five, motif six >= four times, flank length of ssr is 150bp)

2primer_designer: design ssr primer

3filter_primer: filter the primers which have ssr(2-6 basepair,repeats>=4)

4： formatdb the genome file which need analyse and blast the primers to the genome,blast the primer sequence up to the target file, prevent the primer file multi-mapping to the target region. #####sometimes it need to use parameter -b 10000 -v 10000###########

5blast_parse: get the primer which 5' of the primer have 3 mismatch and 3' of primer have 1 mismatch

6generate_primer_pair_new: get the primer that match to the position, extract the pair primers which map to the same scaffold

7final_pair: get the intermediate primer, the product of the pair primer is larger than 1000, you can use the default value or you can define yourself.

8product_ssr_check:get the product by intermediate primer from the given genome

ssr_finder:check ssr in product, repeat numbers of motif should be the same with 1ssr_finder.pl

9filter_ssr: filter the product that contain more than one ssr,one pair primer one ssr

10get_fitprimer: get the suitable primer and ssr(sort by repeat units and repeat numbers) ################################################################## The format in 10get_fitprimer.pl is follow array1: scaffold ID

array2: ssr sequence

array3: ssr units(ssr units length * copys)

array4: the position of the ssr on the scaffold

array5: length of ssr

array6: product of ssr------------ssr_left[ssr]ssr_right

array7: sequences of the forward primer

array8: the suitable amplification temperature for the forward primer

array9: sequences of the reverse primer

array10: the suitable amplification temperature for the reverse primer

array11: length of the amplification product

############################## 
This procedure is to get all ssr from genome. 
After this process, you can define how many 
ssrs on one scaffold (ssr per scaffold), so
you need to think carefully; 

####################################################################
step4 is time-consuming, maybe >=120min, but memory is low,you can run on login nod or you can qsub ####################################################################

#################path is the directory contained perl scripts####################
perl $path/1ssr_finder --flank 150 --ssr2 6 --ssr3 4 --ssr4 4 --ssr5 4 --ssr6 4 $1 $outdir/$2.ssr

perl $path/2primer_designer $outdir/$2.ssr $outdir/$2.raw_primer $outdir/$2.primer_result

perl $path/3filter_primer $outdir/$2.primer_result $outdir/$2.rescreen $outdir/$2.blastin

 formatdb -i $1 -p F
 
 blastall -i $outdir/$2.blastin  -d $1 -p blastn -o $outdir/$2.blast.out -F F -b 10000 -v 10000

perl $path/5blast_parse_change $outdir/$2.blast.out $outdir/$2.query_sbjct 4

perl $path/6generate_primer_pair_new $outdir/$2.query_sbjct $outdir/$2.primer.tab

perl $path/7final_pair $outdir/$2.primer.tab $outdir/$2.rescreen $outdir/$2.inter_primer.tab 2000

perl $path/8product_ssr_check $1 $outdir/$2.inter_primer.tab $outdir/$2.product_file

perl $path/1ssr_finder --ssr2 6 --ssr3 4 --ssr4 4 --ssr5 4 --ssr6 4 $outdir/$2.product_file $outdir/$2.product_ssr

perl $path/9filter_ssr $outdir/$2.product_ssr $outdir/$2.inter_primer.tab $outdir/$2.only_primer.tab

perl $path/10get_fitprimer $outdir/$2.only_primer.tab $outdir/$2.product_ssr $outdir/$2.rescreen $outdir/$2.final_primer
