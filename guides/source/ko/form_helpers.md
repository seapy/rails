[Form Helpers] 폼 헬퍼
============

웹 어플리케이션에서 폼은 사용자의 입력을 위한 필수 인터페이스입니다. 하지만 폼 마크업을 작성하고 수정하는것은 폼 컨트롤의 이름짓기와 많은 속성들로인해 금방 지루해집니다. 레일스는 이러한 복잡한작업을 위해 폼 마크업을 생성하는 뷰헬퍼를 제공합니다. 하지만 다양한 유즈케이스가 있기에 사용하기전에는 헬퍼 메소드의 다른점과 유사점을 알아야할 필요가 있습니다. [[[Forms in web applications are an essential interface for user input. However, form markup can quickly become tedious to write and maintain because of form control naming and their numerous attributes. Rails does away with these complexities by providing view helpers for generating form markup. However, since they have different use-cases, developers are required to know all the differences between similar helper methods before putting them to use.]]]

본 가이드를 읽고나면 다음의 내용들을 이해할 수 있습니다: [[[After reading this guide, you will know:]]]

* 검색폼과 모델에 특정되지 않는 유사한 일반적인 폼의 생성 방법 [[[How to create search forms and similar kind of generic forms not representing any specific model in your application.]]]

* 특정 데이터베이스 레코드를 생성하거나 수정하는 모델중심의 폼 생성 방법. [[[How to make model-centric forms for creation and editing of specific database records.]]]

* 여러종류의 데이터를 표현하는 select 박스 생성 방법. [[[How to generate select boxes from multiple types of data.]]]

* 레일스가 제공하는 날짜, 시간 헬퍼. [[[The date and time helpers Rails provides.]]]

* 파일 업로드 폼을 다르게하는 것. [[[What makes a file upload form different.]]]

* 외부 리소스와 연결하는 폼 생성 방법 [[[Some cases of building forms to external resources.]]]

* 복잡한 폼 생성 방법. [[[How to build complex forms.]]]

--------------------------------------------------------------------------------

NOTE: 본 가이드는 폼 헬퍼와 인수에대한 완전한 문서를 목표로 하지 않습니다. 완전한 문서를 참고하려면 [the Rails API documentation](http://api.rubyonrails.org/) 링크를 방문하세요. [[[This guide is not intended to be a complete documentation of available form helpers and their arguments. Please visit [the Rails API documentation](http://api.rubyonrails.org/) for a complete reference.]]]


[Dealing with Basic Forms] 기본폼 다루기
------------------------

가장 일반적인 폼 헬퍼는 `form_tag` 입니다. [[[The most basic form helper is `form_tag`.]]]

```erb
<%= form_tag do %>
  Form contents
<% end %>
```

인수없이 위와 같이 호출하는경우, `<form>` 태그를 생성하고 전송하는경우 현재 페이지에 POST 요청을 합니다. 예를들어, 현재 페이지가 `/home/index` 인경우 생성되는 HTML은 다음과 같습니다.(가독성을 위해 개행문자가 일부 추가되었습니다.) [[[When called without arguments like this, it creates a `<form>` tag which, when submitted, will POST to the current page. For instance, assuming the current page is `/home/index`, the generated HTML will look like this (some line breaks added for readability):]]]

```html
<form accept-charset="UTF-8" action="/home/index" method="post">
  <div style="margin:0;padding:0">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  </div>
  Form contents
</form>
```

HTML이 몇개의 추가 요소를 가지고 있는것을 확인할 수 있습니다: 2개의 숨겨진 input 요소를 포함한 `div`. 추가된 div 는 중요한데, 이것 없이는 폼이 받아들여지지 않기 때문입니다. 첫번째 input 요소는 `utf8`이라는 이름을 가지고 있으며 폼이 "GET"이나 "POST" 요청을 할때 브라우저가 문자열 인코딩을 제대로 다루도록 합니다. 두번째 input 요소는 `authenticity_token`이라는 이름을 가지고 있으며 레일스에서 **cross-site request forgery protection**라고 부르는 보안기능으로 폼 헬퍼는 GET 요청을 제외한 모든 폼에 생성합니다(이 보안기능이 활성화 되어 있을때 제공). 자세한 내용은 [레일스 어플리케이션 보안](./security.html#cross-site-request-forgery-csrf)을 확인합니다. [[[Now, you'll notice that the HTML contains something extra: a `div` element with two hidden input elements inside. This div is important, because the form cannot be successfully submitted without it. The first input element with name `utf8` enforces browsers to properly respect your form's character encoding and is generated for all forms whether their actions are "GET" or "POST". The second input element with name `authenticity_token` is a security feature of Rails called **cross-site request forgery protection**, and form helpers generate it for every non-GET form (provided that this security feature is enabled). You can read more about this in the [Security Guide](./security.html#cross-site-request-forgery-csrf).]]]

NOTE: 본가이드의 샘플코드에서 `div`의 숨겨진 input 요소는 간결성을 위해 제외됩니다. [[[Throughout this guide, the `div` with the hidden input elements will be excluded from code samples for brevity.]]]

### [A Generic Search Form] 검색 폼

웹에서 가장 기본적인 폼중 하나는 검색 폼입니다. 이 폼은 다음을 포함합니다: [[[One of the most basic forms you see on the web is a search form. This form contains:]]]

* "GET" 메소드를 가진 폼, [[[a form element with "GET" method,]]]

* input을 위한 라벨, [[[a label for the input,]]]

* text input 요소, [[[a text input element, and]]]

* submit 요소. [[[a submit element.]]]

이 폼을 만들기 위해 `form_tag`, `label_tag`, `text_field_tag`, `submit_tag`를 사용해야 할것입니다. 다음과 같은: [[[To create this form you will use `form_tag`, `label_tag`, `text_field_tag`, and `submit_tag`, respectively. Like this:]]]

```erb
<%= form_tag("/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
```

다음과 같은 HTML을 생성합니다: [[[This will generate the following HTML:]]]

```html
<form accept-charset="UTF-8" action="/search" method="get">
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" />
</form>
```

TIP: 모든 폼의 input에 ID 속성값은 name 속성값으로 생성됩니다(예제의 경우 "q"). 이러한 ID들은 CSS 스타일링이나 자바스크립트를 이용한 폼 처리에 매우 유용합니다. [[[For every form input, an ID attribute is generated from its name ("q" in the example). These IDs can be very useful for CSS styling or manipulation of form controls with JavaScript.]]]

`text_field_tag`, `submit_tag` 외에도 HTML의 _모든_ 폼 컨트롤에 대하여 비슷한 헬퍼가 있습니다. [[[Besides `text_field_tag` and `submit_tag`, there is a similar helper for _every_ form control in HTML.]]]

IMPORTANT: 검색 폼에 대해서는 항상 "GET"을 사용합니다. 이렇게 하면 사용자가 특정 검색어를 즐겨찾기해서 다시 찾아올수 있게 합니다. 일반적으로 레일스는 액션에 알맞는 HTTP verb를 사용하도록 권장합니다. [[[Always use "GET" as the method for search forms. This allows users to bookmark a specific search and get back to it. More generally Rails encourages you to use the right HTTP verb for an action.]]]

### [Multiple Hashes in Form Helper Calls] 폼 헬퍼를 호출시 다양한 인수

`form_tag` 헬퍼는 2개의 인수를 받습니다: 액션의 경로와 옵션 해쉬. 이 해쉬는 폼의 속성이나 HTML class와 같은 옵션에 해당합니다. [[[The `form_tag` helper accepts 2 arguments: the path for the action and an options hash. This hash specifies the method of form submission and HTML options such as the form element's class.]]]

`link_to` 헬퍼는 경로 인수는 문자열이 아니어도 됩니다; 레일스 라우팅 매커니즘이 이해하고 알맞은 URL로 변환되는 해쉬도 가능합니다. 하지만 `form_tag`의 경우 경로를 설정할때 두개의 인수를 지정하는경우 문제를 발생시킬수 있습니다. 예를 들어 다음과 같이 적는다면: [[[As with the `link_to` helper, the path argument doesn't have to be a string; it can be a hash of URL parameters recognizable by Rails' routing mechanism, which will turn the hash into a valid URL. However, since both arguments to `form_tag` are hashes, you can easily run into a problem if you would like to specify both. For instance, let's say you write this:]]]

```ruby
form_tag(controller: "people", action: "search", method: "get", class: "nifty_form")
# => '<form accept-charset="UTF-8" action="/people/search?method=get&class=nifty_form" method="post">'
```

`method`와 `class`는 URL의 쿼리문자열에 추가된것을 볼수 있습니다. 2개의 해쉬를 의미하는것이 었다면 하나를 명시해야합니다. 루비에게 첫번째 해쉬인지를(혹은 둘다) 중괄호로 분리해서 알려주어야 합니다. 이는 당신이 예상한 HTML을 만들것입니다: [[[Here, `method` and `class` are appended to the query string of the generated URL because even though you mean to write two hashes, you really only specified one. So you need to tell Ruby which is which by delimiting the first hash (or both) with curly brackets. This will generate the HTML you expect:]]]

```ruby
form_tag({controller: "people", action: "search"}, method: "get", class: "nifty_form")
# => '<form accept-charset="UTF-8" action="/people/search" method="get" class="nifty_form">'
```

### [Helpers for Generating Form Elements] 폼 요소를 생성하는 헬퍼

레일스는 체크박스, 텍스트 필드, 라디오버튼과 같은 폼 요소를 생성하는 일련의 헬퍼를 제공합니다. 이러한 기본 헬퍼는 "_tag"라는 이름으로 끝나고(`text_field_tag`나 `check_box_tag`와 같이), 하나의 `<input>` 요소를 생성합니다. 첫번째 변수는 항상 input의 name입니다. 폼이 전송될때 name은 폼 데이터와 함께 전송되며 사용자의 입력값이 컨트롤러의 `params` 해쉬를 생성합니다. 예를들어 폼에 `<%= text_field_tag(:query) %>`이 있다며 컨트롤러에서 이 필드의 값은 `params[:query]`를 이용해 가져옵니다. [[[Rails provides a series of helpers for generating form elements such as checkboxes, text fields, and radio buttons. These basic helpers, with names ending in "_tag" (such as `text_field_tag` and `check_box_tag`), generate just a single `<input>` element. The first parameter to these is always the name of the input. When the form is submitted, the name will be passed along with the form data, and will make its way to the `params` hash in the controller with the value entered by the user for that field. For example, if the form contains `<%= text_field_tag(:query) %>`, then you would be able to get the value of this field in the controller with `params[:query]`.]]]

input 이름을 지을때 array, hash와 같은 non-scalar 값들을 `params`에서 사용하기위해 레일스는 약간의 관례를 사용합니다. 이에 대한 자세한 것은 [본 가이드의 챕터 7](#understanding-parameter-naming-conventions)를 읽어봅니다. 헬퍼의 정확한 사용방법을 자세히 알고 싶다면 [API documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html)를 참고합니다. [[[When naming inputs, Rails uses certain conventions that make it possible to submit parameters with non-scalar values such as arrays or hashes, which will also be accessible in `params`. You can read more about them in [chapter 7 of this guide](#understanding-parameter-naming-conventions). For details on the precise usage of these helpers, please refer to the [API documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html).]]]

#### [Checkboxes] 체크박스

체크박스는 사용자가 여러개의 옵션을 활성화하거나 비활성화할 수 있도록 하는 폼 컨트롤입니다.: [[[Checkboxes are form controls that give the user a set of options they can enable or disable:]]]

```erb
<%= check_box_tag(:pet_dog) %>
<%= label_tag(:pet_dog, "I own a dog") %>
<%= check_box_tag(:pet_cat) %>
<%= label_tag(:pet_cat, "I own a cat") %>
```

위의 코드는 다음과 같이 생성됩니다: [[[This generates the following:]]]

```html
<input id="pet_dog" name="pet_dog" type="checkbox" value="1" />
<label for="pet_dog">I own a dog</label>
<input id="pet_cat" name="pet_cat" type="checkbox" value="1" />
<label for="pet_cat">I own a cat</label>
```

`check_box_tag`의 첫번째 변수는 당연히 input의 name입니다. 두번째 변수는 input의 값입니다. 이 값은 체크박스가 체크된경우 폼 데이터에 포함됩니다(그리고 `params`에 제공됩니다). [[[The first parameter to `check_box_tag`, of course, is the name of the input. The second parameter, naturally, is the value of the input. This value will be included in the form data (and be present in `params`) when the checkbox is checked.]]]

#### [Radio Buttons] 라디오 버튼

라디오 버튼은 체크박스와 비슷하게 여러개의 옵션을 베타적으로 선택할수 있게하는 폼 컨트롤입니다(예를들어 사용자는 한개만 선택가능): [[[Radio buttons, while similar to checkboxes, are controls that specify a set of options in which they are mutually exclusive (i.e., the user can only pick one):]]]

```erb
<%= radio_button_tag(:age, "child") %>
<%= label_tag(:age_child, "I am younger than 21") %>
<%= radio_button_tag(:age, "adult") %>
<%= label_tag(:age_adult, "I'm over 21") %>
```

결과물: [[[Output:]]]

```html
<input id="age_child" name="age" type="radio" value="child" />
<label for="age_child">I am younger than 21</label>
<input id="age_adult" name="age" type="radio" value="adult" />
<label for="age_adult">I'm over 21</label>
```

`check_box_tag`와 같이 `radio_button_tag`의 두번째 변수는 input의 값입니다. 라디오 버튼은 같은 이름(age)을 공유하고 있기 때문에 사용자는 하나만 선택할 수 있고, `params[:age]`는 "child", "adult"중 하나의 값만 가지게 됩니다. [[[As with `check_box_tag`, the second parameter to `radio_button_tag` is the value of the input. Because these two radio buttons share the same name (age) the user will only be able to select one, and `params[:age]` will contain either "child" or "adult".]]]

NOTE: 체크박스와 라디오버튼에는 항상 라벨을 사용합니다. 특정 옵션과 연결된 텍스트는 클릭가능 영역을 늘려주고 사용자가 input을 쉽게 클릭할 수 있도록 합니다. [[[Always use labels for checkbox and radio buttons. They associate text with a specific option and, by expanding the clickable region, make it easier for users to click the inputs.]]]

### [Other Helpers of Interest] 흥미로운 다른 헬퍼들

textarea, 비밀번호 필드, 숨김 필드, 검색 필드, 전화번호 필드, 날짜 필드, 시간 필드, 색상 필드, datetime-local 필드, month 필드, week 필드, URL 필드, 이메일 필드는 언급할 만한 가치가 있는 폼 컨트롤입니다. [[[Other form controls worth mentioning are textareas, password fields, hidden fields, search fields, telephone fields, date fields, time fields, color fields, datetime fields, datetime-local fields, month fields, week fields, URL fields and email fields:]]]

```erb
<%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
<%= password_field_tag(:password) %>
<%= hidden_field_tag(:parent_id, "5") %>
<%= search_field(:user, :name) %>
<%= telephone_field(:user, :phone) %>
<%= date_field(:user, :born_on) %>
<%= datetime_field(:user, :meeting_time) %>
<%= datetime_local_field(:user, :graduation_day) %>
<%= month_field(:user, :birthday_month) %>
<%= week_field(:user, :birthday_week) %>
<%= url_field(:user, :homepage) %>
<%= email_field(:user, :address) %>
<%= color_field(:user, :favorite_color) %>
<%= time_field(:task, :started_at) %>
```

Output:

```html
<textarea id="message" name="message" cols="24" rows="6">Hi, nice site</textarea>
<input id="password" name="password" type="password" />
<input id="parent_id" name="parent_id" type="hidden" value="5" />
<input id="user_name" name="user[name]" type="search" />
<input id="user_phone" name="user[phone]" type="tel" />
<input id="user_born_on" name="user[born_on]" type="date" />
<input id="user_meeting_time" name="user[meeting_time]" type="datetime" />
<input id="user_graduation_day" name="user[graduation_day]" type="datetime-local" />
<input id="user_birthday_month" name="user[birthday_month]" type="month" />
<input id="user_birthday_week" name="user[birthday_week]" type="week" />
<input id="user_homepage" name="user[homepage]" type="url" />
<input id="user_address" name="user[address]" type="email" />
<input id="user_favorite_color" name="user[favorite_color]" type="color" value="#000000" />
<input id="task_started_at" name="task[started_at]" type="time" />
```

숨김 필드는 사용자에게 보이지 않지만 다른 문자열 input 필드처럼 데이터를 가지고 있습니다. 이 값은 자바스크립트에 의해 변경될 수 있습니다. [[[Hidden inputs are not shown to the user but instead hold data like any textual input. Values inside them can be changed with JavaScript.]]]

IMPORTANT: 검색, 전화번호, 날짜, 시간, 색상, datetime, datetime-local, month, week, URL, 이메일 input은 HTML5 컨트롤입니다. 만약 당신의 앱이 오래된 브라우저를 지원해야 한다면 HTML5 polyfill(CSS 또는 자바스크립트에 의해 제공되는)이 필요할것입니다. 현재 인기 있는 툴 [Modernizr](http://www.modernizr.com/)과 [yepnope](http://yepnopejs.com/)은 감지된 HTML5 기능의 존재 여부에 따라 기능을 추가 할 수있는 간단한 방법을 제공하지만 이것은 확실히 [부족함이 없는 해결 방법](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills)입니다. [[[The search, telephone, date, time, color, datetime, datetime-local, month, week, URL, and email inputs are HTML5 controls. If you require your app to have a consistent experience in older browsers, you will need an HTML5 polyfill (provided by CSS and/or JavaScript). There is definitely [no shortage of solutions for this](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills), although a couple of popular tools at the moment are [Modernizr](http://www.modernizr.com/) and [yepnope](http://yepnopejs.com/), which provide a simple way to add functionality based on the presence of detected HTML5 features.]]]

TIP: 비밀번호 input 필드를 사용한다면(어떠한 목적이던지), 이 변수가 로그에 남지 않도록 어플리케이션 설정을 해야합니다. 자세한 내용은 [Security Guide](security.html#logging)에서 배울수 있습니다. [[[If you're using password input fields (for any purpose), you might want to configure your application to prevent those parameters from being logged. You can learn about this in the [레일스 어플리케이션 보안](security.html#logging).]]]

[Dealing with Model Objects] 모델객체와 연결된 폼 다루기
--------------------------

### [Model Object Helpers] 모델객체 헬퍼

폼의 일반적인 작업은 모델객체를 수정하거나 생성하는것입니다. `*_tag` 헬퍼들은 다소 장황하지만 각 태그들에 알맞은 변수명을 강제하고 적절한 input 기본값을 설정해 이러한 작업에 사용할 수 있습니다. 레일스는 이러한 작업에 맞추어진 헬퍼를 제공합니다. 이러한 헬퍼들은 `text_field`, `text_area` 처럼 _tag 접미사가 제외됩니다. [[[A particularly common task for a form is editing or creating a model object. While the `*_tag` helpers can certainly be used for this task they are somewhat verbose as for each tag you would have to ensure the correct parameter name is used and set the default value of the input appropriately. Rails provides helpers tailored to this task. These helpers lack the _tag suffix, for example `text_field`, `text_area`. ]]]

이 헬퍼들의 첫번째 인수는 인스턴스 변수의 이름이고 두번째 인수는 객체의 메소드 이름(대개는 속성)입니다. 레일스는 객체 메소드의 반환값을 input의 값으로 하고 알맞은 input 이름을 설정합니다. 만약 컨트롤러에 `@person` 변수가 정의되어 있고 사람의 이름이 Henry인 경우 폼은 다음과 같습니다: [[[For these helpers the first argument is the name of an instance variable and the second is the name of a method (usually an attribute) to call on that object. Rails will set the value of the input control to the return value of that method for the object and set an appropriate input name. If your controller has defined `@person` and that person's name is Henry then a form containing:]]]

```erb
<%= text_field(:person, :name) %>
```

다음과 같은 결과를 생성합니다 [[[will produce output similar to]]]

```erb
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

폼 전송시 사용자가 입력한 값은 `params[:person][:name]`에 저장됩니다. `params[:person]` 해쉬는 `Person.new`의 인수 또는 `@person`이 Person 인스턴스인경우 `@person.update`의 인수로 전달하기 알맞습니다. 헬퍼의 두번째 인수로 속성의 이름이 사용되는것이 강제적인것은 아닙니다. 위의 경우 person 객체가 `name` 혹은 `name=` 메소드를 가지고 있다면 레일스의 헬퍼는 동작합니다. [[[Upon form submission the value entered by the user will be stored in `params[:person][:name]`. The `params[:person]` hash is suitable for passing to `Person.new` or, if `@person` is an instance of Person, `@person.update`. While the name of an attribute is the most common second parameter to these helpers this is not compulsory. In the example above, as long as person objects have a `name` and a `name=` method Rails will be happy.]]]

WARNING: `person` 또는 `"person"` 처럼 인스턴스 변수의 이름을 전달해야지 모델 인스턴스를 전달하는것이 아닙니다. [[[You must pass the name of an instance variable, i.e. `:person` or `"person"`, not an actual instance of your model object.]]]

레일스는 모델 객체와 연동된 검증 오류를 표시하는 헬퍼를 제공합니다. 이것들은 [Active Record Validations](./active_record_validations.html#displaying-validation-errors-in-views)가이드에 자세히 설명되어 있습니다. [[[Rails provides helpers for displaying the validation errors associated with a model object. These are covered in detail by the [Active Record Validations](./active_record_validations.html#displaying-validation-errors-in-views) guide.]]]

### [Binding a Form to an Object] 객체에 폼 바인딩

이것은 완벽함과 멀어지면서 편리함을 증가시킵니다. 만약 Person의 많은 속성을 수정하는 경우 변경하려는 객체의 이름을 반복해서 적어야 합니다. 우리가 원하는것은 폼을 모델객체에 바인딩 하는것인데 `form_for`가 정확히 그런 동작을 합니다. [[[While this is an increase in comfort it is far from perfect. If Person has many attributes to edit then we would be repeating the name of the edited object many times. What we want to do is somehow bind a form to a model object, which is exactly what `form_for` does.]]]

articles을 다루는 컨트롤러 `app/controllers/articles_controller.rb`를 가정했을때: [[[Assume we have a controller for dealing with articles `app/controllers/articles_controller.rb`:]]]

```ruby
def new
  @article = Article.new
end
```

`form_for`를 사용하는 뷰 `app/views/articles/new.html.erb`는 다음과 같습니다: [[[The corresponding view `app/views/articles/new.html.erb` using `form_for` looks like this:]]]

```erb
<%= form_for @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

여기 주의할 몇가지 사항이 있습니다: [[[There are a few things to note here:]]]


* `@article`은 수정하려는 실제 객체이다. [[[`@article` is the actual object being edited.]]]

* 옵션은 단일 해쉬이다. 라우팅 옵션은 `:url` 해쉬, HTML 옵션은 `:html` 해쉬에 전달된다. 또한 `:namespace` 옵션을 제공해 폼이 유일한 id 값을 가지게할 수 있다. namespace 속성값은 밑줄문자를 추가후 생성된 HTML id 값에 접두사로 사용된다. [[[There is a single hash of options. Routing options are passed in the `:url` hash, HTML options are passed in the `:html` hash. Also you can provide a `:namespace` option for your form to ensure uniqueness of id attributes on form elements. The namespace attribute will be prefixed with underscore on the generated HTML id.]]]

* `form_for` 메소드는 **폼 빌더** 객체를 yields 한다(`f` 변수). [[[The `form_for` method yields a **form builder** object (the `f` variable).]]]

* 폼 컨트롤을 생성하는 메소드는 폼 빌더 오브젝트 `f`의 **on** 메소드를 호출한다. [[[Methods to create form controls are called **on** the form builder object `f`]]]

HTML 결과는 다음과 같다: [[[The resulting HTML is:]]]

```html
<form accept-charset="UTF-8" action="/articles/create" method="post" class="nifty_form">
  <input id="article_title" name="article[title]" type="text" />
  <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
  <input name="commit" type="submit" value="Create" />
</form>
```

`form_for` 컨트롤에 전달된 name은 `params`의 키로 폼의 값에 접근할 수 있다. `article`의 모든 input은 `article[속성이름]`과 같은 name을 가진다. `create` 액션에서 `params[:article]` 해쉬는 `:title`, `:body` 키를 가진다. parameter_names 섹션에서 보다 자세한 input name에 대한 내용을 알 수 있다. [[[The name passed to `form_for` controls the key used in `params` to access the form's values. Here the name is `article` and so all the inputs have names of the form `article[attribute_name]`. Accordingly, in the `create` action `params[:article]` will be a hash with keys `:title` and `:body`. You can read more about the significance of input names in the parameter_names section.]]]

폼 빌더에 의해 호출된 헬퍼 메소드는 이미 폼 빌더에 의해 관리되어 어떤 객체가 수정되는지 필요하지 않을때를 제외하고는 모델 객체의 헬퍼와 동일하다. [[[The helper methods called on the form builder are identical to the model object helpers except that it is not necessary to specify which object is being edited since this is already managed by the form builder.]]]

`fields_for` 헬퍼를 이용해 실제 `<form>` 태그를 생성하는 대신 바인딩을 생성할 수 있다. 이는 같은 폼 안에서 또 다른 모델객체를 수정할 때 유용하다. 예를들어 만약 Pserson 모델과 연관된 ContactDetail 모델을 가지고 있을때 다음과 같이 둘다 포함하는 폼을 생성할 수 있다: [[[You can create a similar binding without actually creating `<form>` tags with the `fields_for` helper. This is useful for editing additional model objects with the same form. For example if you had a Person model with an associated ContactDetail model you could create a form for creating both like so:]]]

```erb
<%= form_for @person, url: {action: "create"} do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for @person.contact_detail do |contact_details_form| %>
    <%= contact_details_form.text_field :phone_number %>
  <% end %>
<% end %>
```

다음과 같은 결과를 생성한다: [[[which produces the following output:]]]

```html
<form accept-charset="UTF-8" action="/people/create" class="new_person" id="new_person" method="post">
  <input id="person_name" name="person[name]" type="text" />
  <input id="contact_detail_phone_number" name="contact_detail[phone_number]" type="text" />
</form>
```

`fields_for`에 의해 yield된 객체는 `form_for`에 yield된 폼 빌더와 비슷하다(사실 `form_for` 내부에서는 `fields_for`를 호출). [[[The object yielded by `fields_for` is a form builder like the one yielded by `form_for` (in fact `form_for` calls `fields_for` internally).]]]

### [Relying on Record Identification] 레코드 식별에 의지하기

사용자 어플리케이션에서 Article 모델을 사용 하려면 **리소스**에 선언 해야한다. [[[The Article model is directly available to users of the application, so — following the best practices for developing with Rails — you should declare it **a resource**:]]]

```ruby
resources :articles
```

리소스를 선언하는것은 몇가지 사이드 이팩트가 있다. [Rails Routing From the Outside In](routing.html#resource-routing-the-rails-default)에서 리소스 설정과 사용에 대해 보다 자세한 정보를 얻을수 있다. [[[TIP: Declaring a resource has a number of side-affects. See [Rails Routing From the Outside In](routing.html#resource-routing-the-rails-default) for more information on setting up and using resources.]]]

RESTful 리소스를 다룰때, **레코드 식별**에 의지해 `form_for`를 사용하면 상당히 쉬워진다. 모델 인스턴스를 전달하는 것으로 레일스는 모델이름과 rest를 알아낸다. [[[When dealing with RESTful resources, calls to `form_for` can get significantly easier if you rely on **record identification**. In short, you can just pass the model instance and have Rails figure out model name and the rest:]]]

```ruby
## Creating a new article
# long-style:
form_for(@article, url: articles_path)
# same thing, short-style (record identification gets used):
form_for(@article)

## Editing an existing article
# long-style:
form_for(@article, url: article_path(@article), html: {method: "patch"})
# short-style:
form_for(@article)
```

`form_for`의 간략버전은 레코드가 신규인지 기존에 존재하던것인지와 무관하게 동일한 것을 편리하게 한다. 레코드 식별은 `record.new_record?` 메소드를 통해 신규 레코드인지 알아낸다. 또한 정확한 경로에 폼을 전송하고 객체의 클래스에 기반에 이름을 정한다. [[[Notice how the short-style `form_for` invocation is conveniently the same, regardless of the record being new or existing. Record identification is smart enough to figure out if the record is new by asking `record.new_record?`. It also selects the correct path to submit to and the name based on the class of the object.]]]

레일스는 또한 알맞은 `class`, `id`를 자동으로 설정한다: article을 생성하는 폼은 `id`, `class`에 `new_article`를 가진다. 만약 id 23번 article를 수정한다면 `class`는 `edit_article`, `id`는 `edit_article_23`가 된다. 이 속성은 가이드의 간결성을 위해 생략한다. [[[Rails will also automatically set the `class` and `id` of the form appropriately: a form creating an article would have `id` and `class` `new_article`. If you were editing the article with id 23, the `class` would be set to `edit_article` and the id to `edit_article_23`. These attributes will be omitted for brevity in the rest of this guide.]]]

WARNING: STI(단일 테이블 상속)을 모델과 함께 사용한다면, 상위 리소스만 선언된경우 서브클래스는 레코드 식별에 의지할수 없다. 이경우 모델 이름, `:url`, `:method`를 명시해야한다. [[[When you're using STI (single-table inheritance) with your models, you can't rely on record identification on a subclass if only their parent class is declared a resource. You will have to specify the model name, `:url`, and `:method` explicitly.]]]

#### [Dealing with Namespaces] 네임스페이스 다루기

네임스페이스 라우트를 생성하면 `form_for`는 9개의 약칭을 가집니다. 만약 당신의 어플리케이션이 admin 네임스페이스를 가진다면 [[[If you have created namespaced routes, `form_for` has a nifty shorthand for that too. If your application has an admin namespace then]]]

```ruby
form_for [:admin, @article]
```

admin 네임스페이스안의 articles 컨트롤러로 전송하는 폼을 만들게 됩니다(업데이트의 경우 `admin_article_path(@article)`에 전송). 만약 몇개의 네임스페이스 레벨을 가지는 구문은 비슷합니다: [[[will create a form that submits to the articles controller inside the admin namespace (submitting to `admin_article_path(@article)` in the case of an update). If you have several levels of namespacing then the syntax is similar:]]]

```ruby
form_for [:admin, :management, @article]
```

레일스 라우팅 시스템과 관련규칙에 관한 더 자세한 정보는 [routing guide](routing.html)를 참고합니다. [[[For more information on Rails' routing system and the associated conventions, please see the [routing guide](routing.html).]]]


### [How do forms with PATCH, PUT, or DELETE methods work?] 폼의 PATCH, PUT, DELETE 메소드는 어떻게 동작하는가?

레일스 프레임워크는 RESTful 디자인을 장려합니다. 이는 많은 "PATCH", "DELETE" 요청("GET", "POST"외에)을 사용하는것을 의미합니다. 하지만 대부분의 브라우저는 폼을 전송할때 "GET", "POST" 메소드 이외에는 _지원하지 않습니다_. [[[The Rails framework encourages RESTful design of your applications, which means you'll be making a lot of "PATCH" and "DELETE" requests (besides "GET" and "POST"). However, most browsers _don't support_ methods other than "GET" and "POST" when it comes to submitting forms.]]]

레일스는 이 이슈를 POST의 숨겨진 input `"_method"` 이름으로 다른 메소드를 에뮬레이팅하여 원하는 메소드를 반영하도록합니다. [[[Rails works around this issue by emulating other methods over POST with a hidden input named `"_method"`, which is set to reflect the desired method:]]]

```ruby
form_tag(search_path, method: "patch")
```

결과: [[[output:]]]

```html
<form accept-charset="UTF-8" action="/search" method="post">
  <div style="margin:0;padding:0">
    <input name="_method" type="hidden" value="patch" />
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  </div>
  ...
```

POST 데이터를 파싱할때, 레일스는 HTTP 메소드가 내부의 지정된 하나인 경우(예제는 "PATCH") `_method` 특수 파라미터를 고려해 동작하도록 합니다. [[[When parsing POSTed data, Rails will take into account the special `_method` parameter and acts as if the HTTP method was the one specified inside it ("PATCH" in this example).]]]

[Making Select Boxes with Ease] Select 박스 쉽게 만들기
-----------------------------

HTML에서의 Select 박스는 많은 마크업(각각의 선택 항목마다 한개의 `OPTION`)을 필요로 하기 때문에 동적으로 생성하는것이 의미가 있습니다. [[[Select boxes in HTML require a significant amount of markup (one `OPTION` element for each option to choose from), therefore it makes the most sense for them to be dynamically generated.]]]

다음과 같은 마크업이 있습니다: [[[Here is what the markup might look like:]]]

```html
<select name="city_id" id="city_id">
  <option value="1">Lisbon</option>
  <option value="2">Madrid</option>
  ...
  <option value="12">Berlin</option>
</select>
```

여기 유저에게 보여질 도시 이름 목록이 있습니다. 어플리케이션 내부적으로는 옵션의 value 속성에 있는 ID 값만을 사용합니다. 레일스에서 이부분을 어떻게 쉽게 해주는지 살펴 보겠습니다. [[[Here you have a list of cities whose names are presented to the user. Internally the application only wants to handle their IDs so they are used as the options' value attribute. Let's see how Rails can help out here.]]]

### [The Select and Option Tags] Select, Option 태그

가장 일반적인 헬퍼는 `select_tag` 이며, 옵션의 문자열을 감싸는 `SELECT` 태그를 생성합니다. [[[The most generic helper is `select_tag`, which — as the name implies — simply generates the `SELECT` tag that encapsulates an options string:]]]

```erb
<%= select_tag(:city_id, '<option value="1">Lisbon</option>...') %>
```

이것은 시작에 불과하며 option 태그를 동적으로 생성하지는 않습니다. `options_for_select` 헬퍼를 이용해 option 태그를 생성할 수 있습니다: [[[This is a start, but it doesn't dynamically create the option tags. You can generate option tags with the `options_for_select` helper:]]]

```html+erb
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...]) %>

결과: [[[output:]]]

<option value="1">Lisbon</option>
<option value="2">Madrid</option>
...
```

`options_for_select`의 첫번째 인수는 각 항목마다 두개의 항목을 가진 중첩된 배열 입니다: 각 항목은 option 문자열(도시 이름)과 option 값(도시 id)로 이루어졌습니다. option 값은 컨트롤러에 전달됩니다. 이값은 데이터베이스 객체의 id에 상응하는 경우가 보통이지만 상황에따라 아닐수있습니다. [[[The first argument to `options_for_select` is a nested array where each element has two elements: option text (city name) and option value (city id). The option value is what will be submitted to your controller. Often this will be the id of a corresponding database object but this does not have to be the case.]]]

이것을 알면 `select_tag`, `options_for_select`을 이용해 원하는 마크업을 만들수 있습니다: [[[Knowing this, you can combine `select_tag` and `options_for_select` to achieve the desired, complete markup:]]]

```erb
<%= select_tag(:city_id, options_for_select(...)) %>
```

`options_for_select`에 옵션값을 전달해 사전에 선택될 옵션을 지정할 수 있습니다. [[[`options_for_select` allows you to pre-select an option by passing its value.]]]

```html+erb
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...], 2) %>

output:

<option value="1">Lisbon</option>
<option value="2" selected="selected">Madrid</option>
...
```

레일스는 option의 값을 확인해 전달된 값과 일치하면 `selected` 속성을 옵션에 추가합니다. [[[Whenever Rails sees that the internal value of an option being generated matches this value, it will add the `selected` attribute to that option.]]]

TIP: `options_for_select`의 두번째 인수는 내부에서 사용하는 값과 동일해야합니다. 값은 숫자 2인데 인수로 문자열 "2"를 `options_for_select`에 전달할 수 없으며 숫자 2를 전달해야합니다. `params` 해쉬로부터 추출된 값은 모두 문자열이라는것을 유의해야합니다. [[[The second argument to `options_for_select` must be exactly equal to the desired internal value. In particular if the value is the integer 2 you cannot pass "2" to `options_for_select` — you must pass 2. Be aware of values extracted from the `params` hash as they are all strings.]]]

WARNING: `:include_blank` 또는 `:prompt`가 제공되지 않는다면, select의 `required` 속성이 true 인경우 `:include_blank`는 true로 설정되며, `size`는 한개가 되고, `multiple`는 true가 아니게 됩니다. [[[when `:include_blank` or `:prompt` are not present, `:include_blank` is forced true if the select attribute `required` is true, display `size` is one and `multiple` is not true.]]]

해쉬를 이용해 option에 임의의 속성을 추가할 수 있습니다: [[[You can add arbitrary attributes to the options using hashes:]]]

```html+erb
<%= options_for_select([['Lisbon', 1, {'data-size' => '2.8 million'}], ['Madrid', 2, {'data-size' => '3.2 million'}]], 2) %>

결과: [[[output:]]]

<option value="1" data-size="2.8 million">Lisbon</option>
<option value="2" selected="selected" data-size="3.2 million">Madrid</option>
...
```

### [Select Boxes for Dealing with Models] 모델과 연동되는 Select 박스

대부분의 경우 폼 컨트롤은 특정 데이터베이스 모델에 연동되고 레일스는 그 목적을 위한 헬퍼를 제공합니다. 다른 폼 헬퍼와 동일하게 모델과 연동하는경우 `select_tag`에서 `_tag` 접미사를 제거합니다. [[[In most cases form controls will be tied to a specific database model and as you might expect Rails provides helpers tailored for that purpose. Consistent with other form helpers, when dealing with models you drop the `_tag` suffix from `select_tag`:]]]

```ruby
# controller:
@person = Person.new(city_id: 2)
```

```erb
# view:
<%= select(:person, :city_id, [['Lisbon', 1], ['Madrid', 2], ...]) %>
```

세번째 변수(options의 배열)는 `options_for_select`에 전달하는 인수와 동일합니다. 한가지 이점은 사전에 선택될 도시이름에 대해 신경쓰지 않아도 유저가 이미 가지고 있는 도시를 선택합니다 - 레일스는 `@person.city_id` 값으로부터 이를 수행합니다. [[[Notice that the third parameter, the options array, is the same kind of argument you pass to `options_for_select`. One advantage here is that you don't have to worry about pre-selecting the correct city if the user already has one — Rails will do this for you by reading from the `@person.city_id` attribute.]]]

`@person` 영역을 가지는 폼 빌더헬퍼에서 `select` 헬퍼를 사용하고자 한다면 다음과 같습니다: [[[As with other helpers, if you were to use the `select` helper on a form builder scoped to the `@person` object, the syntax would be:]]]

```erb
# select on a form builder
<%= f.select(:city_id, ...) %>
```

WARNING: `belongs_to` association을 설정하기위해 `select`(또는 비슷한 헬퍼인 `collection_select`, `select_tag`)를 사용할때는 assosiation 이름이 아니라 외부키의 이름을 전달해야 합니다.(위의 예제에서는 `city_id`) `city_id`가 아니라 `city`를 사용하면 `params` 해쉬를 `Person.new`나 `update`에 전달할때 Active Record는 ` ActiveRecord::AssociationTypeMismatch: City(#17815740) expected, got String(#1138750) ` 에러를 발생시킵니다. 이를 살펴볼수 있는 또다른 방법은 폼 헬퍼의 속성만을 수정하는것입니다. 사용자가 외부키를 직접 변경하는 잠재적인 보안 문제에 대해 알아야합니다. [[[If you are using `select` (or similar helpers such as `collection_select`, `select_tag`) to set a `belongs_to` association you must pass the name of the foreign key (in the example above `city_id`), not the name of association itself. If you specify `city` instead of `city_id` Active Record will raise an error along the lines of ` ActiveRecord::AssociationTypeMismatch: City(#17815740) expected, got String(#1138750) ` when you pass the `params` hash to `Person.new` or `update`. Another way of looking at this is that form helpers only edit attributes. You should also be aware of the potential security ramifications of allowing users to edit foreign keys directly.]]]

### [Option Tags from a Collection of Arbitrary Objects] 임의의 객체 모음을 위한 option 태그

`options_for_select`를 이용한 option 태그 생성은 각 option의 문자열과 값으로 이루어진 배열을 필요로 합니다. 하지만 City 모델(아마도 Active Record)을 가지고 있고 이들 객체 모음으로부터 option 태그를 생성하고 싶다면 어떻게 해야할까요? 여기에 중첩배열을 만들어내는 한가지 해결방법이 있습니다: [[[Generating options tags with `options_for_select` requires that you create an array containing the text and value for each option. But what if you had a City model (perhaps an Active Record one) and you wanted to generate option tags from a collection of those objects? One solution would be to make a nested array by iterating over them:]]]

```erb
<% cities_array = City.all.map { |city| [city.name, city.id] } %>
<%= options_for_select(cities_array) %>
```

이것은 완벽히 유효한 해결방법이지만 레일스는 간결한 대안인 `options_from_collection_for_select`를 제공합니다. 이 헬퍼는 임의의 객체 모음이 2개의 인수(option의 **value**, **text** 에 접근하는 메서드 이름)를 가지고 있다고 가정합니다: [[[This is a perfectly valid solution, but Rails provides a less verbose alternative: `options_from_collection_for_select`. This helper expects a collection of arbitrary objects and two additional arguments: the names of the methods to read the option **value** and **text** from, respectively:]]]

```erb
<%= options_from_collection_for_select(City.all, :id, :name) %>
```

이름에서 알 수 있듯이, 이것은 option 태그만을 생성합니다. select 박스와 함께 사용하려면 `options_for_select`처럼 `select_tag`와 같이 사용합니다. 모델 객체와 사용하는경우, `select`가 `select_tag`, `options_for_select` 하나로 합친것처럼, `collection_select`는 `select_tag`, `options_from_collection_for_select` 하나로 합친것처럼 동작합니다. [[[As the name implies, this only generates option tags. To generate a working select box you would need to use it in conjunction with `select_tag`, just as you would with `options_for_select`. When working with model objects, just as `select` combines `select_tag` and `options_for_select`, `collection_select` combines `select_tag` with `options_from_collection_for_select`.]]]

```erb
<%= collection_select(:person, :city_id, City.all, :id, :name) %>
```

정리해보면, `options_for_select`가 `select`로 되는것처럼 `options_from_collection_for_select`는 `collection_select`으로 됩니다. [[[To recap, `options_from_collection_for_select` is to `collection_select` what `options_for_select` is to `select`.]]]

NOTE: `options_for_select`에 전달되는 배열의 쌍은 첫번째는 문자열 두번째는 값입니다. 하지만 `options_from_collection_for_select`는 첫번째 인수는 값 메소드, 두번째는 문자열 메소드입니다. [[[Pairs passed to `options_for_select` should have the name first and the id second, however with `options_from_collection_for_select` the first argument is the value method and the second the text method.]]]

### [Time Zone and Country Select] 시간대와 국가 선택

레일스에서 시간대 지원을 사용하려면 사용자에게 어떤 시간대에 있는지 질의해야합니다. 이를 위해 미리 지정되어 있는 TimeZone 객체들을 `collection_select`를 이용해 select option을 생성해야 합니다. 하지만 `time_zone_select` 헬퍼를 이용해 쉽게 사용할 수 있습니다. [[[To leverage time zone support in Rails, you have to ask your users what time zone they are in. Doing so would require generating select options from a list of pre-defined TimeZone objects using `collection_select`, but you can simply use the `time_zone_select` helper that already wraps this:]]]

```erb
<%= time_zone_select(:person, :time_zone) %>
```

또한 좀더 자세한 설정을 위해 `time_zone_options_for_select` 헬퍼가 있습니다. 이 두가지 메소드의 인수에 대한 자세한 내용은 API 문서를 읽어보기 바랍니다. [[[There is also `time_zone_options_for_select` helper for a more manual (therefore more customizable) way of doing this. Read the API documentation to learn about the possible arguments for these two methods.]]]

레일스는 국가를 선택하기위해 `country_select` 헬퍼를 _사용_합니다. 하지만 이것은 [country_select plugin](https://github.com/stefanpenner/country_select)으로 분리되어 있습니다. 이것을 사용할때 특정이름을 목록에 포함하거나 제외하는것은 논란의 여지가 있다는것을 인식해야합니다.(이것은 레일스로부터 분리된 이유이기도 합니다) [[[Rails _used_ to have a `country_select` helper for choosing countries, but this has been extracted to the [country_select plugin](https://github.com/stefanpenner/country_select). When using this, be aware that the exclusion or inclusion of certain names from the list can be somewhat controversial (and was the reason this functionality was extracted from Rails).]]]

[Using Date and Time Form Helpers] 날짜와 시간 폼 헬퍼 사용하기
--------------------------------

HTML5에서 제공하는 날짜와 시간 입력 필드를 생성하는 폼헬퍼를 사용하지 않고 다른 헬퍼를 선택할 수 있습니다. 이러한 날짜와 시간 헬퍼는 다른 폼 헬퍼와 다른 두가지가 있습니다. [[[You can choose not to use the form helpers generating HTML5 date and time input fields and use the alternative date and time helpers. These date and time helpers differ from all the other form helpers in two important respects:]]]

* 날짜와 시간은 하나의 입력 항목으로 표현되지 않습니다. 대신 각 항목의 컴포넌트(년, 월, 일 등...)를 가지며 `params` 해쉬에 하나의 값으로 날짜와 시간이 전달 되지 않습니다. [[[Dates and times are not representable by a single input element. Instead you have several, one for each component (year, month, day etc.) and so there is no single value in your `params` hash with your date or time.]]]

* 다른 헬퍼는 `_tag` 접미사를 가지는 것으로 기본 헬퍼인지 아니면 모델객체와 연결된 객체인지 판단합니다. 하지만 날짜와 시간 헬퍼의 경우 `select_date`, `select_time`, `select_datetime`은 기본 헬퍼, `date_select`, `time_select`, `datetime_select`는 모델객체 헬퍼입니다. [[[Other helpers use the `_tag` suffix to indicate whether a helper is a barebones helper or one that operates on model objects. With dates and times, `select_date`, `select_time` and `select_datetime` are the barebones helpers, `date_select`, `time_select` and `datetime_select` are the equivalent model object helpers.]]]

헬퍼는 여러개의 select 박스로 이루어진 각기 다른 컴포넌트(년도, 월, 일 등)를 생성합니다. [[[Both of these families of helpers will create a series of select boxes for the different components (year, month, day etc.).]]]

### [Barebones Helpers] 기본 헬퍼

`select_*`와 비슷한 헬퍼는 첫번째 인수로 Date, Time, DateTime 인스턴스를 받아서 현재 선택된 값을 결정하는데 사용합니다. 해당 값을 전달하지 않는경우 현재 날짜값이 사용됩니다. 예를들어 [[[The `select_*` family of helpers take as their first argument an instance of Date, Time or DateTime that is used as the currently selected value. You may omit this parameter, in which case the current date is used. For example]]]

```erb
<%= select_date Date.today, prefix: :start_date %>
```

결과(실제 옵션 값은 간결함을 위해 생략) [[[outputs (with actual option values omitted for brevity)]]]

```html
<select id="start_date_year" name="start_date[year]"> ... </select>
<select id="start_date_month" name="start_date[month]"> ... </select>
<select id="start_date_day" name="start_date[day]"> ... </select>
```

위의 input은 `params[:start_date]` 해쉬에 `:year`, `:month`, `:day` 키를 가지도록 합니다. 실제 Time, Date 객체를 얻기 위해서는 추출된 값을 알맞은 생성자에 전달해야 합니다. 예를들어 [[[The above inputs would result in `params[:start_date]` being a hash with keys `:year`, `:month`, `:day`. To get an actual Time or Date object you would have to extract these values and pass them to the appropriate constructor, for example]]]

```ruby
Date.civil(params[:start_date][:year].to_i, params[:start_date][:month].to_i, params[:start_date][:day].to_i)
```

`:prefix` 옵션은 `params` 해쉬에서 날짜 컴포넌트 키로 사용합니다. 여기서는 `start_date`로 설정되어 있고 생략하는경우 기본값은 `date`입니다. [[[The `:prefix` option is the key used to retrieve the hash of date components from the `params` hash. Here it was set to `start_date`, if omitted it will default to `date`.]]]

### [Model Object Helpers] 모델 객체 헬퍼

`select_date`는 `params` 해쉬에 Active Record가 예상하는 적합한 하나의 값으로 제공되지 않기 때문에 Active Record 객체에 수정하거나 생성하는데 알맞지 않습니다. [[[`select_date` does not work well with forms that update or create Active Record objects as Active Record expects each element of the `params` hash to correspond to one attribute.]]]
날짜와 시간을 위한 모델 객체 헬퍼는 특별한 이름을 가진 변수를 전송합니다; Active Record가 보기에 생성자에 적합한 컬럼 타입이 주어지는 형태를 가집니다. 예를들어: [[[The model object helpers for dates and times submit parameters with special names; when Active Record sees parameters with such names it knows they must be combined with the other parameters and given to a constructor appropriate to the column type. For example:]]]

```erb
<%= date_select :person, :birth_date %>
```
결과 (실제 옵션 값은 간결함을 위해 생략) [[[outputs (with actual option values omitted for brevity)]]]

```html
<select id="person_birth_date_1i" name="person[birth_date(1i)]"> ... </select>
<select id="person_birth_date_2i" name="person[birth_date(2i)]"> ... </select>
<select id="person_birth_date_3i" name="person[birth_date(3i)]"> ... </select>
```

`params` 해쉬의 결과는 다음과 같습니다 [[[which results in a `params` hash like]]]

```ruby
{:person => {'birth_date(1i)' => '2008', 'birth_date(2i)' => '11', 'birth_date(3i)' => '22'}}
```

`Person.new` (또는 `update`)에 변수가 전달되면 Active Record는 `birth_date` 속성을 생성하는데 `Date.civil`이 동작하는것처럼 알맞은 값이 전달됩니다. [[[When this is passed to `Person.new` (or `update`), Active Record spots that these parameters should all be used to construct the `birth_date` attribute and uses the suffixed information to determine in which order it should pass these parameters to functions such as `Date.civil`.]]]

### [Common Options] 공통 옵션

두 헬퍼는 각각의 select 태그를 생성하는데 동일한 핵심 함수를 사용하고 많은 비슷한 옵션을 가집니다. 레일스는 현재 년도의 앞뒤 5년에 해당하는 년도 옵션을 생성합니다. 만약 이게 적절한 범위가 아니라면 `:start_year`, `:end_year` 옵션을 이용해 변경할 수 있습니다. 사용가능한 옵션의 완전한 목록은 [API 문서](http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html)를 참고하시기 바랍니다. [[[Both families of helpers use the same core set of functions to generate the individual select tags and so both accept largely the same options. In particular, by default Rails will generate year options 5 years either side of the current year. If this is not an appropriate range, the `:start_year` and `:end_year` options override this. For an exhaustive list of the available options, refer to the [API documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html).]]]

경험으로 볼때 모델 객체와 연동할때는 `date_select`를 사용하고 그렇지 않고 검색 제한과 같은 다른경우에는 `select_date`를 사용하는것이 좋습니다. [[[As a rule of thumb you should be using `date_select` when working with model objects and `select_date` in other cases, such as a search form which filters results by date.]]]

NOTE: 많은 경우 브라우저 자체 날짜 선택창은 어설프고 날짜와 한주의 시작일이 연동되는것이 고려되지 않습니다. [[[In many cases the built-in date pickers are clumsy as they do not aid the user in working out the relationship between the date and the day of the week.]]]

### [Individual Components] 개별 컴포넌트

가끔 년도나 월처럼 하나의 날짜 컴포넌트만 표시할 필요가 있습니다. 레일스는 이를 위해 `select_year`, `select_month`, `select_day`, `select_hour`, `select_minute`, `select_second` 헬퍼를 제공합니다. 이 헬퍼들은 쉽게 사용할 수 있습니다. 기본값으로 컴포넌트의 이름으로 input 이름을 설정 하고(예를들어 `select_year`는 "year", `select_month`는 "month") 이는 `:field_name` 옵션으로 변경 가능 합니다. `:prefix` 옵션은 `select_date`, `select_time`에서와 기본값, 동작박식이 동일합니다. [[[Occasionally you need to display just a single date component such as a year or a month. Rails provides a series of helpers for this, one for each component `select_year`, `select_month`, `select_day`, `select_hour`, `select_minute`, `select_second`. These helpers are fairly straightforward. By default they will generate an input field named after the time component (for example "year" for `select_year`, "month" for `select_month` etc.) although this can be overridden with the  `:field_name` option. The `:prefix` option works in the same way that it does for `select_date` and `select_time` and has the same default value.]]]

첫번째 변수는 선택될 날짜로 Date, Time, DateTime 인스턴스이거나 컴포넌트에 적절한 값이거나 숫자입니다. 예를들어 [[[The first parameter specifies which value should be selected and can either be an instance of a Date, Time or DateTime, in which case the relevant component will be extracted, or a numerical value. For example]]]

```erb
<%= select_year(2009) %>
<%= select_year(Time.now) %>
```

현재 년도가 2009년이라면 동일한 결과를 생성하고 유저가 선택한 값은 `params[:date][:year]`에서 찾을수 있습니다. [[[will produce the same output if the current year is 2009 and the value chosen by the user can be retrieved by `params[:date][:year]`.]]]

[Uploading Files] 파일 업로드 
---------------

사람의 사진이나 작업할 내용을 포함한 CSV 파일과 같은 것이든 파일을 업로드 하는것은 일반적인 작업입니다. 파일을 업로드 하는데 기억해야할 가장 중요한것은 form 인코딩이 **반드시** "multipart/form-data" 이어야 한다는것입니다. `form_for`를 사용하는경우 이는 자동으로 적용됩니다. `form_tag`를 사용하는경우 다음 예제와 같이 직접 설정해야합니다. [[[A common task is uploading some sort of file, whether it's a picture of a person or a CSV file containing data to process. The most important thing to remember with file uploads is that the rendered form's encoding **MUST** be set to "multipart/form-data". If you use `form_for`, this is done automatically. If you use `form_tag`, you must set it yourself, as per the following example.]]]

다음 2개의 form은 파일을 업로드 합니다. [[[The following two forms both upload a file.]]]

```erb
<%= form_tag({action: :upload}, multipart: true) do %>
  <%= file_field_tag 'picture' %>
<% end %>

<%= form_for @person do |f| %>
  <%= f.file_field :picture %>
<% end %>
```

레일스는 두개의 헬퍼를 제공합니다: 기본헬퍼인 `file_field_tag`, 모델과 연동된 `file_field` 헬퍼. 다른 헬퍼들과 다르게 기본값을 설정할 수 없다는것이 유일하게 다른점입니다. 첫번째 예제의 업로드 파일은 `params[:picture]`에 두번째 예제는 `params[:person][:picture]`에 전달될것을 예상할 수 있습니다. [[[Rails provides the usual pair of helpers: the barebones `file_field_tag` and the model oriented `file_field`. The only difference with other helpers is that you cannot set a default value for file inputs as this would have no meaning. As you would expect in the first case the uploaded file is in `params[:picture]` and in the second case in `params[:person][:picture]`.]]]

### [What Gets Uploaded] 업로드된것은 어떻게 가져오는가

`params` 해쉬에 저장된 업로드된 객체는 IO의 서브클래스 인스턴스입니다. 업로드되는 파일 사이즈에 따라서 StringIO 혹은 임시 저장된 파일의 File 인스턴스가 됩니다. 두 경우 모두 `original_filename` 속성에 사용자 컴퓨터의 파일이름을 가지고 `content_type` 속성에 업로드된 파일의 MIME 종류가 설정됩니다. 다음의 코드는 업로드된 객체를 `#{Rails.root}/public/uploads`에 원본파일과 동일한 이름으로 저장합니다.(form은 이전 예제라고 가정합니다) [[[The object in the `params` hash is an instance of a subclass of IO. Depending on the size of the uploaded file it may in fact be a StringIO or an instance of File backed by a temporary file. In both cases the object will have an `original_filename` attribute containing the name the file had on the user's computer and a `content_type` attribute containing the MIME type of the uploaded file. The following snippet saves the uploaded content in `#{Rails.root}/public/uploads` under the same name as the original file (assuming the form was the one in the previous example).]]]

```ruby
def upload
  uploaded_io = params[:person][:picture]
  File.open(Rails.root.join('public', 'uploads', uploaded_io.original_filename), 'w') do |file|
    file.write(uploaded_io.read)
  end
end
```

파일이 업로드되면 이미지 리사이징, 썸네일 생성을 위한 파일의 저장위치(디스크, 아마존 S3 등)와 모델객체의 연결과 같은 여러가지의 잠재적 작업이 있습니다. 이러한 작업은 본 가이드의 범위를 벗어나지만 이러한 작업을 위한 라이브러리들이 있습니다. [CarrierWave](https://github.com/jnicklas/carrierwave)와 [Paperclip](http://www.thoughtbot.com/projects/paperclip)이 가장 잘 알려진것들입니다. [[[Once a file has been uploaded, there are a multitude of potential tasks, ranging from where to store the files (on disk, Amazon S3, etc) and associating them with models to resizing image files and generating thumbnails. The intricacies of this are beyond the scope of this guide, but there are several libraries designed to assist with these. Two of the better known ones are [CarrierWave](https://github.com/jnicklas/carrierwave) and [Paperclip](http://www.thoughtbot.com/projects/paperclip).]]]

NOTE: 사용자가 파일을 선택하지 않으면 이에 상응하는 파라미터에는 빈 문자열이 설정됩니다. [[[If the user has not selected a file the corresponding parameter will be an empty string.]]]

### [Dealing with Ajax] Ajax로 다루기

다른 form들과 다르게 비동기적인 파일 업로드는 `form_for`에서 제공하는 `remote: true`로 간단히 되지 않습니다. Ajax form 직렬화는 브라우저안의 자바스크립트에 의해서 실행되는데 자바스크립트는 하드 드라이브에 있는 파일을 읽을수 없기 때문에 업로드 할 수 없습니다. 가장 일반적인 해결책은 보이지 않는 iframe를 이용해 form을 전송하는것입니다. [[[Unlike other forms making an asynchronous file upload form is not as simple as providing `form_for` with `remote: true`. With an Ajax form the serialization is done by JavaScript running inside the browser and since JavaScript cannot read files from your hard drive the file cannot be uploaded. The most common workaround is to use an invisible iframe that serves as the target for the form submission.]]]

[Customizing Form Builders] Customizing Form Builders
-------------------------

이전에 언급한것처럼 `form_for`, `fields_for`의 yield된 객체는 FormBuilder의 인스턴스(혹은 상속받은 클래스의 인스턴스) 입니다. form 빌더는 한개의 객체를 위한 form 요소의 출력을 캡슐화한것입니다. 당신의 form을 위해 헬퍼를 만들수도 있고, FormBuilder를 상속받고 헬퍼를 추가할 수 있습니다. [[[As mentioned previously the object yielded by `form_for` and `fields_for` is an instance of FormBuilder (or a subclass thereof). Form builders encapsulate the notion of displaying form elements for a single object. While you can of course write helpers for your forms in the usual way, you can also subclass FormBuilder and add the helpers there. For example]]]

```erb
<%= form_for @person do |f| %>
  <%= text_field_with_label f, :first_name %>
<% end %>
```

다음과 같이 대체 가능 [[[can be replaced with]]]

```erb
<%= form_for @person, builder: LabellingFormBuilder do |f| %>
  <%= f.text_field :first_name %>
<% end %>
```

LabellingFormBuilder 클래스는 다음과 같은 형태로 정의 [[[by defining a LabellingFormBuilder class similar to the following:]]]

```ruby
class LabellingFormBuilder < ActionView::Helpers::FormBuilder
  def text_field(attribute, options={})
    label(attribute) + super
  end
end
```

만약 이를 자주 재사용하게 된다면 `labeled_form_for` 헬퍼를 만들어 자동으로 `builder: LabellingFormBuilder` 옵션이 적용되게 할 수 있습니다. [[[If you reuse this frequently you could define a `labeled_form_for` helper that automatically applies the `builder: LabellingFormBuilder` option.]]]

form 빌더는 다음과 같은 상황에서도 어떤일은 할지 결정하는데 사용됩니다. [[[The form builder used also determines what happens when you do]]]

```erb
<%= render partial: f %>
```

만약 `f`가 FormBuilder 인스턴스라면 `form` partial을 사용하고 partial의 object 변수에 form 빌더를 설정합니다. form 빌더가 LabellingFormBuilder의 인스턴스인경우 `labelling_form` partial을 사용합니다. [[[If `f` is an instance of FormBuilder then this will render the `form` partial, setting the partial's object to the form builder. If the form builder is of class LabellingFormBuilder then the `labelling_form` partial would be rendered instead.]]]

[Understanding Parameter Naming Conventions] 파라미터 이름 규칙에 대한 이해
------------------------------------------

이전 섹션에서 살펴 본것처럼 form으로부터 전송받은 값들은 `params` 해쉬 혹은 그 하위에 중첩 해쉬형태로 저장됩니다. 예를들어 Person 모델의 `create` 액션은 `params[:person]`에 person 인스턴스를 생성하기 위한 모든 속성값이 저장되어 있습니다. `params` 해쉬는 배열, 해쉬들의 배열등도 가질수 있습니다. [[[As you've seen in the previous sections, values from forms can be at the top level of the `params` hash or nested in another hash. For example in a standard `create` action for a Person model, `params[:person]` would usually be a hash of all the attributes for the person to create. The `params` hash can also contain arrays, arrays of hashes and so on.]]]

기본적으로 HTML form은 구조화된 데이터에 대해 알지 못하고 단순한 문자열인경우 모두 이름-값 형태로 생성됩니다. 어플리케이션에서 배열과 해쉬를 사용하기위해서는 레일스의 이름 규칙에 따른 결과입니다. [[[Fundamentally HTML forms don't know about any sort of structured data, all they generate is name–value pairs, where pairs are just plain strings. The arrays and hashes you see in your application are the result of some parameter naming conventions that Rails uses.]]]

TIP: 다음의 예제들은 Racks 파라미터 파서를 이용해 콘솔에서 빠르게 확인할 수있습니다. 예를들어, [[[You may find you can try out examples in this section faster by using the console to directly invoke Racks' parameter parser. For example,]]]

```ruby
Rack::Utils.parse_query "name=fred&phone=0123456789"
# => {"name"=>"fred", "phone"=>"0123456789"}
```

### [Basic Structures] 기본 자료구조

두개의 기본 자료구조는 배열과 해쉬입니다. 해쉬는 `params` 값에 접근하는 방법과 동일한 규칙을 가집니다. 예를들어 form이 다음과 같다면 [[[The two basic structures are arrays and hashes. Hashes mirror the syntax used for accessing the value in `params`. For example if a form contains]]]

```html
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

`params` 해쉬는 다음과 같습니다 [[[the `params` hash will contain]]]

```erb
{'person' => {'name' => 'Henry'}}
```

그리고 컨트롤러에서는 `params[:person][:name]`와 같이 전송된 값을 조회할 수 있습니다. [[[and `params[:person][:name]` will retrieve the submitted value in the controller.]]]

해쉬는 원하는만큼 중첩될수 있습니다. 예를들어 [[[Hashes can be nested as many levels as required, for example]]]

```html
<input id="person_address_city" name="person[address][city]" type="text" value="New York"/>
```

`params` 해쉬는 다음과 같습니다 [[[will result in the `params` hash being]]]

```ruby
{'person' => {'address' => {'city' => 'New York'}}}
```

일반적으로 레일스에서는 중복되는 파라미터 이름은 무시합니다. 만약 파라미터 이름이 빈 대괄호[]로 이루어진경우 배열로 저장됩니다. 만약 people에 여러개의 phone_number가 존재하는 경우, form에서 다음과 같이할 수 있습니다. [[[Normally Rails ignores duplicate parameter names. If the parameter name contains an empty set of square brackets [] then they will be accumulated in an array. If you wanted people to be able to input multiple phone numbers, you could place this in the form:]]]

```html
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
```

이에 대한 결과는 `params[:person][:phone_number]`에 배열로 저장됩니다. [[[This would result in `params[:person][:phone_number]` being an array.]]]

### [Combining Them] 조합해서 사용

우리는 두개의 컨셉을 적절히 섞어서 사용할 수 있습니다. 이전의 예제에서 한개의 해쉬 항목이 배열이 될수도 있고, 해쉬의 배열이 될수도 있습니다. 예를들어 여러개의 주소를 가지는 form조각이 반복되는 form을 만들수 있습니다. [[[We can mix and match these two concepts. For example, one element of a hash might be an array as in the previous example, or you can have an array of hashes. For example a form might let you create any number of addresses by repeating the following form fragment]]]

```html
<input name="addresses[][line1]" type="text"/>
<input name="addresses[][line2]" type="text"/>
<input name="addresses[][city]" type="text"/>
```

이에대한 결과로 `params[:addresses]`는 배열이 되고 배열의 항목은 `line1`, `line2`, `city` 키를 가진 해쉬로 이루어집니다. 레일스는 이미 존재하는 해쉬와 동일한 이름이 입력되면 새로운 해쉬를 생성합니다. [[[This would result in `params[:addresses]` being an array of hashes with keys `line1`, `line2` and `city`. Rails decides to start accumulating values in a new hash whenever it encounters an input name that already exists in the current hash.]]]

하지만 해쉬의 중첩에는 한개 레벨의 배열만 가질수 있다는 제약이 있습니다. 배열은 대개 해쉬로 대체가능합니다. 예를들어 모델 객체의 배열대신 모델객체의 id, 배열의 인덱스, 다른 파라미터를 키로하는  하나의 해쉬로 대체 가능합니다. [[[There's a restriction, however, while hashes can be nested arbitrarily, only one level of "arrayness" is allowed. Arrays can be usually replaced by hashes, for example instead of having an array of model objects one can have a hash of model objects keyed by their id, an array index or some other parameter.]]]

WARNING: 배열 파라미터는 `check_box` 헬퍼에 대해서 잘 동작하지 않습니다. HTML 스펙정의에 보면 체크되지 않은 checkbox는 값을 전송하지 않습니다. 하지만 보통 편의를 위해 checkbox의 값을 항상 전송합니다. `check_box` 헬퍼는 이를위해 동일한 이름을 가지는 hidden input을 만들어 처리합니다. checkbox가 체크되지 않은경우 hidden input의 값만 전송되고 체크된경우는 두개 모두 전송되지만 checkbox의 값을 우선사용합니다. 배열 파라미터를 이와같이 중복되게 전송하는경우 레일스는 언제 새로운 배열을 만들어야될지 결정하는데 혼란이옵니다. `check_box_tag`를 사용하거나 배열대신 해쉬를 사용하는것이 더 좋습니다. [[[Array parameters do not play well with the `check_box` helper. According to the HTML specification unchecked checkboxes submit no value. However it is often convenient for a checkbox to always submit a value. The `check_box` helper fakes this by creating an auxiliary hidden input with the same name. If the checkbox is unchecked only the hidden input is submitted and if it is checked then both are submitted but the value submitted by the checkbox takes precedence. When working with array parameters this duplicate submission will confuse Rails since duplicate input names are how it decides when to start a new array element. It is preferable to either use `check_box_tag` or to use hashes instead of arrays.]]]

### [Using Form Helpers] Form 헬퍼 사용

이전 섹션에서는 레일스 form 헬퍼 전부를 사용하지 않았습니다. input name을 직접 만들어 `text_field_tag`와 같이 헬퍼에 직접전달할때 레일스는 보다 높은 수준의 도움을 제공합니다. 당신의 name 파라미터를 처리를 위해 `form_for`, `fields_for` 두개의 헬퍼의 `:index` 옵션을 이용합니다. [[[The previous sections did not use the Rails form helpers at all. While you can craft the input names yourself and pass them directly to helpers such as `text_field_tag` Rails also provides higher level support. The two tools at your disposal here are the name parameter to `form_for` and `fields_for` and the `:index` option that helpers take.]]]

당신은 각 사람마다 여러개의 주소를 가지는 form을 렌더링할수 있습니다. 예를들어: [[[You might want to render a form with a set of edit fields for each of a person's addresses. For example:]]]

```erb
<%= form_for @person do |person_form| %>
  <%= person_form.text_field :name %>
  <% @person.addresses.each do |address| %>
    <%= person_form.fields_for address, index: address do |address_form|%>
      <%= address_form.text_field :city %>
    <% end %>
  <% end %>
<% end %>
```

한 사람이 2개의 주소를 가진다고 가정하고, id는 23, 45라면 출력은 다음과 같을것입니다: [[[Assuming the person had two addresses, with ids 23 and 45 this would create output similar to this:]]]

```html
<form accept-charset="UTF-8" action="/people/1" class="edit_person" id="edit_person_1" method="post">
  <input id="person_name" name="person[name]" type="text" />
  <input id="person_address_23_city" name="person[address][23][city]" type="text" />
  <input id="person_address_45_city" name="person[address][45][city]" type="text" />
</form>
```

`params` 해쉬의 결과는 다음과 같습니다 [[[This will result in a `params` hash that looks like]]]

```ruby
{'person' => {'name' => 'Bob', 'address' => {'23' => {'city' => 'Paris'}, '45' => {'city' => 'London'}}}}
```

레일스는 form 빌더로부터 `fields_for`가 호출되었기 때문에 이러한 input들이 person 해쉬의 일부라는것을 알고 있습니다. `:index` 옵션은 레일스에게 `person[address][city]` 대신 배열을 의미하는 []로 address와 city 사이를 감싸라고 알립니다. 만약 Active Record 객체를 전달한다면 레일스는 `to_param`을 호출하고 기본값으로 데이터베이스의 id를 리턴합니다. 이는 수정해야할 Address를 알아내는데 유용합니다. 중요한 숫자나 문자열, `nil`을 전달할 수 있습니다(배열 파라미터 결과에 나타낼 값). [[[Rails knows that all these inputs should be part of the person hash because you called `fields_for` on the first form builder. By specifying an `:index` option you're telling Rails that instead of naming the inputs `person[address][city]` it should insert that index surrounded by [] between the address and the city. If you pass an Active Record object as we did then Rails will call `to_param` on it, which by default returns the database id. This is often useful as it is then easy to locate which Address record should be modified. You can pass numbers with some other significance, strings or even `nil` (which will result in an array parameter being created).]]]

보다 복잡한 중첩을 생성하기 위해 input name의 첫번째 부분(이전 예제의 `person[address]`)을 명시할 수 있습니다. 예를들어 [[[To create more intricate nestings, you can specify the first part of the input name (`person[address]` in the previous example) explicitly, for example]]]

```erb
<%= fields_for 'person[address][primary]', address, index: address do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

다음과 같은 결과를 생성합니다 [[[will create inputs like]]]

```html
<input id="person_address_primary_1_city" name="person[address][primary][1][city]" type="text" value="bologna" />
```

일반적인 규칙으로 결과의 input name은 `fields_for`/`form_for`에 주어진 name, index 값, input의 name이 추가된 형태입니다. `:index` 옵션을 `text_field`와 같은 헬퍼에 직접 전달할수도 있지만 개별 input에 지정하기보다는 일반적으로 반복적인 작업을 줄이기위해 form 빌더 레벨에 지정합니다. [[[As a general rule the final input name is the concatenation of the name given to `fields_for`/`form_for`, the index value and the name of the attribute. You can also pass an `:index` option directly to helpers such as `text_field`, but it is usually less repetitive to specify this at the form builder level rather than on individual input controls.]]]

손쉬운 방법으로 name에 []를 추가해 `:index` 옵션을 제거할 수 있습니다. 이는 `index: address` 옵션과 동일합니다 [[[As a shortcut you can append [] to the name and omit the `:index` option. This is the same as specifying `index: address` so]]]

```erb
<%= fields_for 'person[address][primary][]', address do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

생성된 결과는 이전의 예제와 동일합니다. [[[produces exactly the same output as the previous example.]]]

[Forms to external resources] 외부 리소스 Form
---------------------------

외부 리소스에 데이터를 전송해야할 필요가 있는 경우에도 레일스 폼 헬퍼를 사용하는것이 좋습니다. 하지만 가끔 `authenticity_token` 값을 설정할 필요가 있습니다. `form_tag` 옵션에 `authenticity_token: '외부리소스 토큰'` 파라미터를 전달해 이를 설정합니다: [[[If you need to post some data to an external resource it is still great to build your form using rails form helpers. But sometimes you need to set an `authenticity_token` for this resource. You can do it by passing an `authenticity_token: 'your_external_token'` parameter to the `form_tag` options:]]]

```erb
<%= form_tag 'http://farfar.away/form', authenticity_token: 'external_token') do %>
  Form contents
<% end %>
```

외부 결제 게이트웨이와 같이 외부 리소스에 데이터를 전송하는데 있어서 때때로 외부 API에 의해 사용할 수 있는 필드가 제한되기도 합니다. 따라서 `authenticity_token` 히든 필드를 생성할 필요가 없을수도 있습니다. 이를 위해 `:authenticity_token` 옵션에 `false`를 전달합니다: [[[Sometimes when you submit data to an external resource, like payment gateway, fields you can use in your form are limited by an external API. So you may want not to generate an `authenticity_token` hidden field at all. For doing this just pass `false` to the `:authenticity_token` option:]]]

```erb
<%= form_tag 'http://farfar.away/form', authenticity_token: false) do %>
  Form contents
<% end %>
```

`form_for`에 대해서도 동일: [[[The same technique is also available for `form_for`:]]]

```erb
<%= form_for @invoice, url: external_url, authenticity_token: 'external_token' do |f| %>
  Form contents
<% end %>
```

마찬가지로 `authenticity_token` 필드를 생성하고 싶지 않을때: [[[Or if you don't want to render an `authenticity_token` field:]]]

```erb
<%= form_for @invoice, url: external_url, authenticity_token: false do |f| %>
  Form contents
<% end %>
```

[Building Complex Forms] 복잡한 폼 만들기
----------------------

많은 앱들이 간단한 폼에서 하나의 객체를 수정하는것을 넘어서 커집니다. 예를들어 Person 객체를 만들때 사용자가 동일한 폼에서 여러개의 주소(집, 직장 등)를 추가하거나 나중에 person을 수정할때 사용자가 필요에 따라 주소를 추가, 삭제, 수정 할 수 있게 합니다. [[[Many apps grow beyond simple forms editing a single object. For example when creating a Person you might want to allow the user to (on the same form) create multiple address records (home, work, etc.). When later editing that person the user should be able to add, remove or amend addresses as necessary.]]]

### [Configuring the Model] 모델 설정

Active Record는 모델 레벨에서 `accepts_nested_attributes_for` 메소드를 제공합니다: [[[Active Record provides model level support  via the `accepts_nested_attributes_for` method:]]]

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses
end

class Address < ActiveRecord::Base
  belongs_to :person
end
```

이는 `Person` 모델에 `addresses_attributes=` 메소드를 생성하고 addresses를 추가, 수정, 삭제할 수 있게 합니다. [[[This creates an `addresses_attributes=` method on `Person` that allows you to create, update and (optionally) destroy addresses.]]]

### [Building the Form] 폼 만들기

다음의 폼은 사용자가 `Person`을 생성하고 addresses와 관계를 가지도록 합니다. [[[The following form allows a user to create a `Person` and its associated addresses.]]]

```html+erb
<%= form_for @person do |f| %>
  Addresses:
  <ul>
    <%= f.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>

        <%= addresses_form.label :street %>
        <%= addresses_form.text_field :street %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

중복 속성을 허용한 경우 `fields_for`는 관계에 해당하는 각 항목에 대해 블록이 한번씩 렌더링합니다. person이 addresses를 하나도 가지지 않는경우 아무것도 렌더링되지 않습니다. 일반적으로 컨트롤러에서 한개 이상의 비어있는 자식객체를 만들어서 적어도 한개의 필드세트가 사용자에게 보여지게 합니다. 아래의 예제는 person 폼을 렌더링할때 3개의 주소 필드 세트가 추가됩니다. [[[When an association accepts nested attributes `fields_for` renders its block once for every element of the association. In particular, if a person has no addresses it renders nothing. A common pattern is for the controller to build one or more empty children so that at least one set of fields is shown to the user. The example below would result in 3 sets of address fields being rendered on the new person form.]]]

```ruby
def new
  @person = Person.new
  3.times { @person.addresses.build}
end
```

`fields_for`는 폼빌더를 만들때 파라미터 이름을 `accepts_nested_attributes_for`에 의해 얻게됩니다. 예를들어 2개의 주소를 가진 객체를 생성하면 전송된 파라미터는 다음과 같습니다. [[[`fields_for` yields a form builder that names parameters in the format expected the accessor generated by `accepts_nested_attributes_for`. For example when creating a user with 2 addresses, the submitted parameters would look like]]]

```ruby
{
    :person => {
        :name => 'John Doe',
        :addresses_attributes => {
            '0' => {
                :kind  => 'Home',
                :street => '221b Baker Street',
            },
            '1' => {
                :kind => 'Office',
                :street => '31 Spooner Street'
            }
        }
    }
}
```

`:addresses_attributes` 해쉬의 키는 그저 다른 주소를 얻는데 필요할 뿐 중요하지 않습니다. [[[The keys of the `:addresses_attributes` hash are unimportant, they need merely be different for each address.]]]

만약 associated 객체가 이미 저장된 상태라면 `fields_for`는 자동으로 저장된 객체의 `id`를 숨겨진 input으로 생성합니다. 이를 비활성화 시키려면 `fields_for`의 옵션에 `include_id: false`를 설정합니다. 자동생성된 input의 위치가 유효하지 않은 HTML이 되거나 사용하는 ORM이 id를 가지지 않을때 필요할 것입니다. [[[If the associated object is already saved, `fields_for` autogenerates a hidden input with the `id` of the saved record. You can disable this by passing `include_id: false` to `fields_for`. You may wish to do this if the autogenerated input is placed in a location where an input tag is not valid HTML or when using an ORM where children do not have an id.]]]

### [The Controller] 컨트롤러

모델에 데이터를 전달하기전에 컨트롤러의 [파라미터 화이트리스트](action_controller_overview.html#strong-parameters)에 추가해야합니다. [[[As usual you need to [whitelist the parameters](action_controller_overview.html#strong-parameters) in the controller before you pass them to the model:]]]

```ruby
def create
  @person = Person.new(person_params)
  # ...
end

private
def person_params
  params.require(:person).permit(:name, addresses_attributes: [:id, :kind, :street])
end
```

### [Removing Objects] 객체 삭제

사용자가 associated 객체를 삭제 하도록 하기 위해서는 `accepts_nested_attributes_for` 옵션에 `allow_destroy: true`를 추가합니다. [[[You can allow users to delete associated objects by passing `allow_destroy: true` to `accepts_nested_attributes_for`]]]

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses, allow_destroy: true
end
```

객체의 해쉬가 `_destroy` 키에 '1' 혹은 'true' 값인경우 객체는 삭제됩니다. 다음의 폼은 사용자가 주소를 삭제할 수 있게 합니다. [[[If the hash of attributes for an object contains the key `_destroy` with a value of '1' or 'true' then the object will be destroyed. This form allows users to remove addresses:]]]

```erb
<%= form_for @person do |f| %>
  Addresses:
  <ul>
    <%= f.fields_for :addresses do |addresses_form| %>
      <li>
        <%= check_box :_destroy%>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

컨트롤러의 params 화이트리스트에 `_destroy` 필드를 추가하는것을 잊지 말아야합니다: [[[Don't forget to update the whitelisted params in your controller to also include the `_destroy` field:]]]

```ruby
def person_params
  params.require(:person).
    permit(:name, addresses_attributes: [:id, :kind, :street, :_destroy])
end
```

### [Preventing Empty Records] 빈 데이터 방지

보통 사용자가 채우지 않은 필드들을 무시하는것은 유용합니다. `accepts_nested_attributes_for` 에 `:reject_if` proc를 전달해서 동작 방식을 변경할 수 있습니다. proc는 폼에의해 전송된 해쉬마다 호출됩니다. 만약 proc가 `false`를 반환하면 Active Record는 해당 해쉬에 대해 associated 객체를 생성하지 않습니다. 아래의 예제는 address의 `kind` 속성이 설정된 경우만 생성합니다. [[[It is often useful to ignore sets of fields that the user has not filled in. You can control this by passing a `:reject_if` proc to `accepts_nested_attributes_for`. This proc will be called with each hash of attributes submitted by the form. If the proc returns `false` then Active Record will not build an associated object for that hash. The example below only tries to build an address if the `kind` attribute is set.]]]

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses, reject_if: lambda {|attributes| attributes['kind'].blank?}
end
```

편의를 위해 `:all_blank` 심볼을 전달하는것으로 대체되는데 이는 `_destroy` 속성을 제외한 모든 속성이 비어있는경우 해당 해쉬가 거부되는 proc를 생성합니다. [[[As a convenience you can instead pass the symbol `:all_blank` which will create a proc that will reject records where all the attributes are blank excluding any value for `_destroy`.]]]

### [Adding Fields on the Fly] 동적 필드 추가 

여러개의 필드 세트를 미리 렌더링하는것이 아니라 사용자가 'Add new child' 버튼을 클릭했을때 추가하는것을 원할 수 있습니다. 하지만 레일스는 이에 대한 지원을 하지 않습니다. 필드 세트를 생성할때 associated 해쉬의 키가 유일하도록 해야합니다. - 자바스크립트의 현재시간이(epoch이후 경과한 밀리세컨드) 일반적인 선택 [[[Rather than rendering multiple sets of fields ahead of time you may wish to add them only when a user clicks on an 'Add new child' button. Rails does not provide any builtin support for this. When generating new sets of fields you must ensure the the key of the associated array is unique - the current javascript date (milliseconds after the epoch) is a common choice.]]]
