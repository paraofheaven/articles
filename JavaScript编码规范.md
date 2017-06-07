## �ѷ���ƽ̨ǰ��JavaScript����淶���

#### ����淶

���Ŷ�Э�������У�����淶��������ز��ö�˵��������Ҫ�������ѡ����Ӧ�ĸ������ߣ����̶��ϱ�֤����������

##### ѡ���׼

+ �Ͽɶȸߣ��ù淶�����Ѿ����߼�����Ϊ���� JavaScript ��׼��
+ ֧����Ŀ�ļ���ѡ��
+ �걸�Ĳ��֧��

�ο����ϼ�����׼�� airbnb �Ŷ��� GitHub ά���� [airbnb/javascript](https://github.com/airbnb/javascript) ˲����ӱ������

+ Ŀǰ GitHub �������ȶȵĹ淶���пֲ��� 35000+ �� star
+ ֧�� ES6��React ��
+ �ٷ��ṩ ESlint��JSCS ���֧��

##### ��������

airbnb �Ŷ��ڷ���������ͬʱ�����ṩ����֮��Ӧ�� eslint �����ļ��������ṩ�������ļ�������ʹ�û�����ͬ�������֣�

1 `eslint-config-airbnb`  

֧��`ECMAScript 6+`��`React`

- `npm install --save-dev eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint`

- add `"extends" : "airbnb"` to your `.eslintrc` 

2 `eslint-config-airbnb-base`

֧��`ECMAScript 6+`

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`

- add `"extends": "airbnb-base"` to your `.eslintrc`

3 `eslint-config-airbnb-base/legacy`

֧��ES5

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`

- add `"extends": "airbnb-base/legacy"` to your `.eslintrc`

##### Eslint Parser

��������л���ES5��д��������ʹ��`base-eslint`��Ϊeslint��eslint��parser,ֻҪ��`.eslintrc`�м������´��뼴�ɡ�

```js
"parser": "babel-eslint"
```

##### ���ӻ���ʾ

����webstorm�ṩ��ESlint�������`Setting\Languages&Frameworks\JavaScript\Code Quality Tools\ESlint `ѡ��Enbale���ɣ�webstorm���Զ�����`.eslintrc`��`pacjage.json`������js�ļ�����lint�������⣬�������Զ���eslintConfig�ļ�����Additional rules ��

webstorm���˿��Խ���lint�������������Զ���һЩ�򵥵Ĵ���;�������޸����һ�ѡ��`fix ESlint problems`���������Դ����������޸�����淶��ʱ�䣬���ò�˵��webstorm����ǰ�˿�������������
