# week1

## 2장 의미 있는 이름

### 의도를 분명히 밝혀라

- 메소드 뿐만 아니라 변수명(특히 리스트 변수명)도 의미를 분명하게 하여 코드를 더욱 명확하게 하자
- 검색하기 쉬운 이름을 사용하자
- 클래스 이름이나 객체 이름은 명사나 명사구가 적합하다.
- 메서드 이름은 동사나 동사구가 적합하다. 접근자, 변경자, 조건자는 javabean 표준에 따라 값 앞에 get, set, is를 붙인다. 생성자를 중복정의할 때는 `정적 팩터리 메서드`를 사용하여 인수를 설명하는 이름을 붙이자.

### 리팩토링하며 적용하자

- 메서드 명: `searchFragranceListByIngredient` → `searchFragranceByIngredient` : list와 같은 불필요한 맥락 생략하였다.
- result와 index를 ingredientOfTheDayGroup과 order로 수정하면서 의미를 분명히 하였고 getTodayIngredient()를 getIngredientsOfTheDay로 수정하며 일관성을 고려하였다.

    ```java
    public IngredientOfTheDay recommendIngredient() {
            List<IngredientOfTheDay> result = ingredientRepository.getTodayIngredient();

            int index = LocalDate.now().getDayOfYear() % 5; // 오늘 날짜 기준으로 인덱스 생성

            return result.get(index);
    }
    ```

    ↓

    ```java
    public IngredientOfTheDay recommendIngredient() {
            List<IngredientOfTheDay> ingredientOfTheDayGroup = ingredientRepository.getIngredientsOfTheDay();

            int order = LocalDate.now().getDayOfYear() % 5; // 오늘 날짜 기준으로 순서 확인
            return ingredientOfTheDayGroup.get(order); // 순서에 해당하는 향료 리턴
    }
    ```

- 원래 dto 이름을 DisplayBrand, DisplayFamily, DisplayIngredient, DisplayScrap 등등 나름 유기적으로 명명했으나 클래스 이름은 명사나 명사구가 적합하다하여 다음과 같이 수정하였다.

    DisplayBrand, DisplayFamily, DisplayIngredient

    → BrandDetailsDto, FamilyDetailsDto, IngredientDetailsDto

    : 각각 브랜드, 향료, 계열의 자세한 설명, 백그라운드 이미지, 키워드 등을 담고 있으며 BrandDto와 같은 명명도 고려해보았으나 비교적 간소한 속성들만 담아내는 dto도 존재할 것 같아 Details를 추가하였다.
