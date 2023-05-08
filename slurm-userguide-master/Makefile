FILE=slurm-userguide
FILES=slurm-userguide.tex slurm.tex
FILE=slurm-slide
FILES=slurm-slide.tex slurm.tex
$(FILE).pdf:$(FILES)
	xelatex -E $(FILE).tex
#	gbk2uni $(FILE).out
#	latex -E $(FILE).tex
#	dvips $(FILE).dvi
#	ps2pdf $(FILE).ps

scp:
#	rsync -vau cfetr-userguide.pdf root@cfetr:/opt/document/cfetr高性能计算集群使用指南.pdf
	rsync -vau slurm-userguide.pdf scc:web/doc/userguide/
clean:
	rm -f *.dvi *.log *.toc *.aux $(FILE).out
