# リファクタノックメモ
## package.json
```json:問題のあるpackage.json
"dependencies": {
    "@types/node": "^20",
}"
```
```json:修正後のpackage.json
"devDependencies": {
    "@types/node": "^20",
}"
```

## 
