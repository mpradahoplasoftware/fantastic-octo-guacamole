# Comandos útiles shell scripts

## Comandos útiles AWK

### Filtrar errores del MTK

`mtk_DW_STG_20200221022857.log |grep -i ": error" > errors1.tmp cat errors1.tmp |grep -v "^DB-" |grep -v "^2020" |awk 'NR%2{printf "%s ",$0;next;}1' > errores2.tmp cat errors1.tmp |grep -v "^DB-" |grep -v "^2020" |awk 'NR%2{printf "%s ",$0;next;}1' |sed s/"com.edb.MTKException:"//g |sed s/"com.edb.util.PSQLException:"//g > errores3.tmp`

### No mostrar la columna 1 y 2

`cat mtk_DW_STG_20200221022857.log |grep "^2020" |grep MTK- |awk '{$1=$2=""; print $0}'`

