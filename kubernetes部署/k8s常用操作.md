> 本文作者：丁辉

# 批量删除pv或pvc

## 删除pvc
```
for line in  $(kubectl get pvc -n $YOURNAMESPACE | awk ' NR > 1{print $1}') ;do  kubectl delete pvc  $line  -n $YOURNAMESPACE ; echo $line; done
```

## 删除pv
```
for line in  $(kubectl get pv -n $YOURNAMESPACE | awk ' NR > 1{print $1}') ;do  kubectl delete pv  $line  -n $YOURNAMESPACE ; echo $line; done
```

## 若发现pv删除不了
```
for line in  $(kubectl get pv -n $YOURNAMESPACE | awk ' NR > 1{print $1}') ;do  kubectl patch pv $line -p '{"metadata":{"finalizers":null}}' -n $YOURNAMESPACE ; kubectl delete pv  $line  -n $YOURNAMESPACE ;echo $line; done
```

