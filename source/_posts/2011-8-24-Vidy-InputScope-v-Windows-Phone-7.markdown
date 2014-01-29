---
layout: post
title: Виды InputScope в Windows Phone 7
date: Wed, 24 Aug 2011 15:37:00 +0000
comments: true
---

Windows Phone 7 позволяет разработчикам в зависимости от их целей показывать различные раскладки софтварной клавиатуры пользователям. Реализуется это c помощью свойства **InputScope.**

<TextBox Height="300" Width="440" VerticalAlignment="Top" InputScope="Text" />

К сожалению при выборе свойства InputScope Intellisense не работает. Для того чтобы устранить эту неприятность достаточно использовать следующий код:

``` xml Использование Input Scope
<TextBox Height="300" Width="440" VerticalAlignment="Top">
	<TextBox.InputScope>
		<InputScope>
			<InputScopeName NameValue="Text"/>
		</InputScope>
	</TextBox.InputScope>
</TextBox>
```

<!--more--> 


В таком случае вы увидите все возможные варианты свойства InputScope (коих больше 60):

1. AddressCity
2. AddressCountryName
3. AddressCountryShortName
4. AddressStateOrProvince
5. AddressStreet
6. AlphanumericFullWidth
7. AlphanumericHalfWidth
8. ApplicationEnd
9. Bopomofo
10. Chat
11. CurrencyAmount
12. CurrencyAmountAndSymbol
13. CurrencyChinese
14. Date
15. DateDay
16. DateDayName
17. DateMonth
18. DateMonthName
19. DateYear
20. Default
21. Digits
22. EmailNameOrAddress
23. EmailSmtpAddress
24. EmailUserName
25. EnumString
26. FileName
27. FullFilePath
28. Hanja
29. Hiragana
30. KatakanaFullWidth
31. KatakanaHalfWidth
32. LogOnName
33. Maps
34. NameOrPhoneNumber
35. Number
36. NumberFullWidth
37. OneChar
38. Password
39. PersonalFullName
40. PersonalGivenName
41. PersonalMiddleName
42. PersonalNamePrefix
43. PersonalNameSuffix
44. PersonalSurname
45. PhraseList
46. PostalAddress
47. PostalCode
48. Private
49. RegularExpression
50. Search
51. Srgs
52. TelephoneAreaCode
53. TelephoneCountryCode
54. TelephoneLocalNumber
55. TelephoneNumber
56. Text
57. Time
58. TimeHour
59. TimeMinorSec
60. Url
61. Xml
62. Yomi

