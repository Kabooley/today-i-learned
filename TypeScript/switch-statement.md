# swtich statement in TypeScript

## 検査対象がunion typeのとき

次の`ts.forEachChild`は`ts.Node`型をコールバックへ引数として渡すが、

この`ts.Node`はあらゆる型の基底クラスであるため、

引数`node`は`ts.Node`を基底としたクラスのオブジェクトになりうる。

そのため、

ts.Node <-- ts.Statement < -- ts.ImportDeclaration
ts.Node <-- ts.Statement < -- ts.ExportDeclaration  

という派生関係のため以下のように

`node: ts.ImportDeclaration | ts.ExportDeclaration | ts.Node`

という型を与えたが...

`ts.ImportDeclaration`, `ts.ExportDeclaration`のメソッド`moduleSpecifier`は``ts.Node`にはないですエラーが出る。

```TypeScript
const getRequireStatements = (title: string, code: string) => {
  const requires: string[] = [];

  const sourceFile: ts.SourceFile = ts.createSourceFile(
    title,
    code,
    ts.ScriptTarget.Latest,
    true,
    ts.ScriptKind.TS
  );

  ts.forEachChild(sourceFile, (
    node: ts.ImportDeclaration | ts.ExportDeclaration | ts.Node
  ) => {
    switch (node.kind) {
      case ts.SyntaxKind.ImportDeclaration: {
        requires.push(node.moduleSpecifier.getText());
        break;
      }

      case ts.SyntaxKind.ExportDeclaration: {
        // For syntax 'export ... from '...'''
        if (node.moduleSpecifier) {
          requires.push(node.moduleSpecifier.getText());
        }
        break;
      }
      default: {
        /* */
      }
    }
  });

  return requires;
};
```

次のようにする

```TypeScript
const getRequireStatements = (title: string, code: string) => {
  const requires: string[] = [];

  const sourceFile: ts.SourceFile = ts.createSourceFile(
    title,
    code,
    ts.ScriptTarget.Latest,
    true,
    ts.ScriptKind.TS
  );

  // node might have its type as ImportDeclaration or ExportDeclaration. 
  // Inherritance:   
  // ts.Node <-- ts.Statement < -- ts.ImportDeclaration  
  // ts.Node <-- ts.Statement < -- ts.ExportDeclaration  
  // 
  ts.forEachChild(sourceFile, (
    node: ts.ImportDeclaration | ts.ExportDeclaration | ts.Node
  ) => {


    switch (node.kind) {
      case ts.SyntaxKind.ImportDeclaration: {
        // NOTE: 解決その一
        requires.push((node as ts.ImportDeclaration).moduleSpecifier.getText());
        // NOTE: 解決その二
        requires.push((<ts.ImportDeclaration>node).moduleSpecifier.getText());
        break;
      }

      case ts.SyntaxKind.ExportDeclaration: {
        // NOTE:　解決その一
        const n: ts.ExportDeclaration = node as ts.ExportDeclaration;
        // For syntax 'export ... from '...'''
        if (n.moduleSpecifier) {
          requires.push(n.moduleSpecifier.getText());
        }
        break;
      }
      default: {
        /* */
      }
    };
  });

  return requires;
};

```

参考：

https://stackoverflow.com/questions/50774790/switch-for-specific-type-in-typescript

