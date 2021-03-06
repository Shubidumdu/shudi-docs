# Custom Element CheckList

> 출처 : [Google Developers](https://developers.google.com/web/fundamentals/web-components/best-practices?authuser=0)

커스텀 요소들은 HTML을 확장하여 본인 스스로의 태그를 갖게 해준다. 해당 기능은 어마어마하지만, 저수준의 기능이기도 해서, 어떻게 활용하는 것이 제일 좋은지 불명확한 경우가 많다.

커스텀 요소를 최적으로 활용하기 위해 해당 체크리스트를 확인하자. 제대로 동작하는 커스텀 요소들을 구성하는 내용들을 나눈 것이다.

## 체크리스트

### **Shadow DOM**

- 스타일을 캡슐화하기 위해 섀도우 루트를 생성하라

  > 섀도우 루트에 스타일링을 캡슐화 시키는 것은 어디에서 해당 요소가 사용되든 해당 스타일링이 적용될 것을 보장한다.

- 섀도우 루트는 `constructor` 내에서 생성하라

  > `constuctor`는 요소 본인에 관한 지식들을 보관하는 곳이다. 따라서 여기에는 다른 요소들을 활용하지 않는 구현 디테일들을 설정하기 적절하다. <br/> 만약, `connectedCallback`에서 이런 내용들을 수행하면, 요소가 분리/연결되는 경우의 상황을 고려해야 한다.

- 커스텀 요소가 생성하는 하위 요소들은 섀도우 루트 안에 넣어라

  > 커스텀 요소에 의해 생성된 자식들은 `private`해야한다. 만약 섀도우 루트의 보호가 없다면, 해당 자식 요소들은 외부 JS에 의해 간섭받을 수 있다.

- light DOM에서의 자식들을 반영하기 위해 `<slot>`을 사용해라

  > `<slot>`을 활용하면 커스텀 요소가 담고 있는 요소들을 이용자들이 지정하기 편하게 만들 수 있다.

- 기본적으로 `inline`으로 설정된 스타일링을 원하는 게 아니라면, `:host`의 `display` 스타일을 변경하라.

> 기본적으로 커스텀요소는 `display: inline` 설정을 갖는다. 따라서 단순히 `width`나 `height`를 설정하는 것은 아무 영향도 없다. 만약, 애초에 `inline` 디스플레이 설정을 의도하는 게 아니라면, 적절히 변경하라.

- `hidden` 속성에 대응하기 위한 `:host` 디스플레이 스타일을 추가하라
  > 섀도우 루트에서 `:host`로 스타일링을 하게되면 이는 HTML 자체적인 `hidden` 속성을 덮어씌우게 된다. 때문에, `:host([hidden]) { display: none }`와 같은 식으로, `hidden`속성을 갖고 있는 경우에 대해 적절한 스타일링 처리가 필요하다.

### **속성(Attributes)와 프로퍼티**

- 글로벌 속성(global attributes)들을 덮어쓰지(override) 말아라
  > 글로벌 속성들은 모든 HTML 요소들에 존재하는 것이다. 예를 들면 `tabindex`와 `role`이 이에 해당한다. 커스텀 요소가 기본적으로 `tabindex`를 0으로 초기화하게끔 설정하고 싶을 수도 있다. 그러나 항상 해당 커스텀 요소를 사용하는 개발자가 이를 다른 값으로 설정할 수 있음을 유의해라. 때문에 아래와 같은 체크가 필요하다.

```js
connectedCallback() {
  if (!this.hasAttribute('role'))
    this.setAttribute('role', 'checkbox');
  if (!this.hasAttribute('tabindex'))
    this.setAttribute('tabindex', 0);
```

- 항상 원시(primitive) 데이터들을 속성 혹은 프로퍼티 모두로 가져올 수 있게 하라.

  > 커스텀 요소들은 수정가능해야 한다. 그리고 이러한 수정은 속성 혹은 프로퍼티 어느쪽으로든 적절히 이루어질 수 있어야 한다. 결국, 이상적으로 모든 원시 속성들은 프로퍼티와 연결되어 있어야 한다.

- 원시 데이터 속성과 프로퍼티들을 항상 동기화(sync)시키도록 해라. 프로퍼티는 속성에 반영되어야하고, 반대도 마찬가지다.

  > 요소를 활용하는 사람들이 어떤 식으로 해당 요소와 상호작용 할지는 알 수 없다. 때문에 속성과 프로퍼티가 서로를 항상 반영하도록 해야한다. 물론 예외도 존재한다. 비디오 플레이어의 `currentTime`과 같은 너무 변경 빈도가 잦은 프로퍼티는 매번 속성에 반영하는 것이 부적절하다.

- Object, Array와 같은 리치 데이터(rich data)들은 프로퍼티로만 받아와라

  > 사실,애초에 내장 HTML 요소에서 속성을 통해 이러한 류의 데이터를 받아들이는 예시 자체가 없다. 대신에 이런 데이터들은 메서드 호출이나 프로퍼티를 통해서 전달된다. 만약, 굳이 이들을 속성으로 전달하고자 하는 경우, 명확한 단점들이 몇가지 있다. 1) 거대한 객체를 문자열로 직렬화(Serialize)하는데에 너무 많은 비용이 들고, 2) 또한 이 문자열화(Stringify) 과정에서 객체에 대한 참조가 사라질 수도 있다.

- 요소를 업그레이드하기 이전에, 이미 설정되었을지도 모르는 프로퍼티를 체크해봐라

  > 커스텀 요소를 활용하는 개발자들이 해당 요소를 불러오기 이전에 먼저 프로퍼티를 설정할지도 모른다. 이런 상황은 종종 로딩 컴포넌트를 핸들링하거나, 해당 컴포넌트를 페이지에 찍어내거나, 해당 프로퍼티를 모델에 바인딩하는 프레임워크를 사용하거나 할 때 종종 발생한다.

- 클래스를 자동으로 적용시키지 마라
  > 요소들은 본인의 상태를 속성을 통해서 나타내야 한다. `class` 속성을 해당 요소를 사용하는 개발자들에 의한 것으로 간주되어야 하며, 이를 임의로 자동으로 설정하는 경우 개발자들의 `class` 관리를 망쳐버릴 수 있다.

### **Events**

- 내부 컴포넌트 활동에 따라 적절히 이벤트를 디스패치하라
  > 오직 컴포넌트 본인만 알 수 있는 활동이 있을 수 있다. 이를테면 타이머나 애니메이션 완료, 혹은 로딩이 완료되는 시점과 같은 것들이다. 이러한 변화에 따라, 호스트에게 해당 컴포넌트의 상태가 변경되었음을 알려주게끔 이벤트를 전달하는 것이 좋다.
- 프로퍼티 설정에 대해서는 별도로 이벤트를 디스패치할 필요없다.
  > 호스트가 프로퍼티를 설정한 내용에 대해 이벤트를 전달하는 것은 불필요하다. 호스트가 직접 설정한 내용이기 때문에 현재 상태를 직접 인지할 수 있기 때문이다. 또한, 호스트가 프로퍼티를 설정한 것에 대한 반응으로 이벤트를 전달하는 경우, 데이터 바인딩과 함께 무한 루프를 유발할 수 있다.

## **Explainers**

### 프로퍼티를 Lazy하게 만들어라

개발자가 커스텀 요소를 불러오기 전에 먼저 프로퍼티를 설정하고자 할 수도 있다. 이는 로딩 컴포넌트를 다루는 프레임워크 등에서 특히 이루어진다.

아래 예시에서는, Angular가 `isChecked` 프로퍼티를 체크박스의 `checked` 프로퍼티에 바인딩하려고 한다. 만약 해당 커스텀 요소가 **lazy-load**된다면 Angular는 요소가 업그레이드되기 이전에 먼저 `checked`프로퍼티를 설정할 수 있을 것이다.

```html
<howto-checkbox [checked]="defaults.isChecked"></howto-checkbox>
```

커스텀 요소는 본인의 인스턴스에 어떤 요소가 이미 설정되어 있는지에 대해 확인함으로써 이러한 경우를 다룰 수 있다. 아래에서 `_upgradeProperty()` 메서드가 그러한 역할을 한다.

```js
connectedCallback() {
  ...
  this._upgradeProperty('checked');
}

_upgradeProperty(prop) {
  if (this.hasOwnProperty(prop)) {
    let value = this[prop];
    delete this[prop];
    this[prop] = value;
  }
}
```

`_upgradeProperty()`는 업그레이드되지 않은 인스턴스로부터 값을 가져온 후, 프로퍼티를 삭제하여 커스텀 요소가 자체적인 프로퍼티 `setter`를 사용하지 않도록 만든다. 이를 통해, 커스텀 요소가 최종적으로 로드되었을 때, 곧바로 수정된 상태를 반영할 수 있도록 만든다.

### 재방문 이슈(reentrancy issues)를 피해라

`attributeChangeCallback()`을 사용하여 상태를 기본 프로퍼티에 반영되도록 하자.

```js
// When the [checked] attribute changes, set the checked property to match.
attributeChangedCallback(name, oldValue, newValue) {
  if (name === 'checked')
    this.checked = newValue;
}
```

헌데, 프로퍼티 설정자가 속성에도 반영되는 경우 무한 루프를 만들어내는 문제가 발생한다.

```js
set checked(value) {
  const isChecked = Boolean(value);
  if (isChecked)
    // OOPS! This will cause an infinite loop because it triggers the
    // attributeChangedCallback() which then sets this property again.
    this.setAttribute('checked', '');
  else
    this.removeAttribute('checked');
}
```

이에 대한 대안으로, 프로퍼티에 대한 setter와 getter를 모두 만들어 getter가 속성에 따라 값을 결정하도록 할 수 있다.

```js
set checked(value) {
  const isChecked = Boolean(value);
  if (isChecked)
    this.setAttribute('checked', '');
  else
    this.removeAttribute('checked');
}

get checked() {
  return this.hasAttribute('checked');
}
```

이제, 속성을 삭제하거나 추가하는 작업은 프로퍼티에도 영향을 미칠 것이다.

끝으로, `attributeChangedCallback()`는 ARIA 상태를 적용하는 것과 같은 사이드 이펙트를 처리하는 데에 사용해라.

```JS
attributeChangedCallback(name, oldValue, newValue) {
  const hasValue = newValue !== null;
  switch (name) {
    case 'checked':
      // Note the attributeChangedCallback is only handling the *side effects*
      // of setting the attribute.
      this.setAttribute('aria-checked', hasValue);
      break;
    ...
  }
}
```
