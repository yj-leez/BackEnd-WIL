# Ch3 함수

### 작게 만들어라!

- 함수는 20줄도 길다.
- if문/ else문/ while문 등에 들어가는 블록은 한 줄이어야 한다. 즉, 거기서 함수를 호출해야한다. 블록 안에서 호출하는 함수 이름을 적절히 짓는다면, 코드를 이해하기도 쉬워진다.

### 한 가지만 해라!

- 함수는 추상화 수준에서 하나의 단계를 처리하도록 짜되, 의미를 유지하기 위해 노력해야한다. 쉽게 말하자면, 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하고 있는 셈이다.
- 코드는 위에서 아래로 이야기처럼 읽히도록, 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 오도록 하자.

### 서술적인 이름을 사용하라!

- 코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다.
- 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.

함수 인수

- 함수에서 이상적인 인수 개수는 0개이며 다음은 1개, 다음은 2개이다. 가능한 3개는 피하는 편이 좋고, 4개 이상은 특별한 이유가 필요하다.
- 많이 쓰는 단항 형식으로는 인수에 질문을 던지는 경우(예를 들면 `boolean fileExists(”MyFile”)`)과 인수를 뭔가로 변환해 결과를 반환하는 경우(예를 들면 `InputStream fileOpen(”MyFile”)`)이다.

플래그 인수

- 함수로 boolean 값을 넘기는 것은 추천하지 않는다. 왜냐하면 함수가 한꺼번에 여러 가지를 처리한다고 공표하는 것이기 때문이다.

인수 객체

- 인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어 본다.

```sql
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

### 함수를 어떻게 짜죠?

- 처음에는 길고 복잡해도 괜찮으나 서투른 코드를 빠짐없이 테스트하는 단위 테스트 케이스를 만든다. 그런 다음 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거하며 메서드를 줄이고 순서를 바꾸며 때론 전체 클래스를 쪼개기도 한다. 하지만 그 중에도 항상 단위 테스트는 통과해야한다.

### 리팩토링하며 적용하자

```java
public IngredientDetailsDto getIngredientContent(String ingredient) {
        // 향료가 존재하는지 확인 후 조회
        if(!ingredientRepository.existsByKorName(ingredient)) throw new CustomException(ErrorCode.INGREDIENT_NOT_FOUND);
        Ingredient findIngredient = ingredientRepository.findByKorName(ingredient);

        // 향료에 해당하는 향료 타입이 존재하는지 확인 후 조회
        IngredientType findIngredientType = ingredientTypeRepository.findById(findIngredient.getIngredientType().getId())
                .orElseThrow(()-> new CustomException(ErrorCode.INGREDIENT_TYPE_NOT_FOUND));

        return IngredientDetailsDto.from(findIngredient, findIngredientType);
}
```

↓ 불필요하게 IngredientType까지 조회해서 두 개의 인자로 넘겨줬던 함수를 조회도 삭제하고 Ingredient만 넘겨주도록 수정하였다.

```java
public IngredientDetailsDto getIngredientContent(String ingredient) {
        // 향료가 존재하는지 확인 후 조회
        if(!ingredientRepository.existsByKorName(ingredient)) throw new CustomException(ErrorCode.INGREDIENT_NOT_FOUND);
        Ingredient findIngredient = ingredientRepository.findByKorName(ingredient);

        return IngredientDetailsDto.from(findIngredient);
}

public static IngredientDetailsDto from(Ingredient ingredient){
        return new IngredientDetailsDto(ingredient.getKorName(),
                ingredient.getEngName(),
                ingredient.getAcode(),
                ingredient.getKeyword(),
                ingredient.getSummary(),
                ingredient.getBackgroundImg(),
                ingredient.getIngredientType().getKorName(),
                ingredient.getIngredientType().getIcon());
}
```

---

```java
public PageableResponse searchFragranceList(FragranceFilterCond cond, PageRequest pageRequest){
        Pageable pageable = pageRequest.of();

        String additionalFamily = null; // 추가 계열 변수 초기화
        // family에 값이 들어온 경우 값 처리
        if(hasText(cond.getFamily())) {
            String[] parts = cond.getFamily().split("\\s+", 2);
            cond.setFamily(parts.length > 0 ? parts[0] : null);
            additionalFamily = parts.length > 1 ? parts[1] : null;
        }

        Page<DisplayFragrance> result = fragranceFamilyRepository.searchByFilter(cond, additionalFamily, pageable); // 향수 조회
        return new PageableResponse(result.getContent(), result.getTotalPages(), result.getTotalElements());

    }
```

↓ FragranceFilterCond에 컬럼을 추가한 뒤 Condition 객체를 파싱하는 함수를 분리하였다.

```java
public PageableResponse searchFragranceByBrandAndFamily(FragranceFilterCond cond, PageRequest pageRequest){
        Pageable pageable = pageRequest.of();
        parseFilterCond(cond);

        Page<FragranceCatalogDto> result = fragranceFamilyRepository.searchByBrandAndFamily(cond, pageable); // 향수 조회
        return new PageableResponse(result.getContent(), result.getTotalPages(), result.getTotalElements());
    }

    private FragranceFilterCond parseFilterCond(FragranceFilterCond cond){
        if(hasText(cond.getFamily())) {
            String[] parts = cond.getFamily().split("\\s+", 2);
            cond.setFamily(parts.length > 0 ? parts[0] : null);
            cond.setAdditionalFamily(parts.length > 1 ? parts[1] : null);
        }

        return cond;
    }
```

그 외에도 메서드 명 변경, 함수 분리 등을 적용할 수 있었다.

✔️ 궁금한 점: 함수형으로 에러 처리 하는 것과 try/catch 블록으로 에러 처리 하는 것의 차이점이 뭘까

<aside>
💡 GPT의 답 :

1. **가시성 (Visibility):**
    - `try/catch`: 예외가 발생하면 해당 블록 내에서만 처리되며, 이 블록 외부에서는 예외의 영향을 직접적으로 확인할 수 없습니다.
    - `함수형 에러 처리`: 예외가 값으로 표현되기 때문에 이 값을 함수 호출자에게 반환하고, 호출자는 명시적으로 에러를 처리하거나 전파할 수 있습니다.
2. **부수 효과 (Side Effects):**
    - `try/catch`: 예외 처리 블록 내에서 상태 변경 등의 부수 효과가 발생할 수 있습니다.
    - `함수형 에러 처리`: 값으로 예외를 다루기 때문에 부수 효과 없이 예외를 다룰 수 있습니다.
3. **코드 흐름 (Control Flow):**
    - `try/catch`: 코드의 흐름이 예외가 발생하면 예외 블록으로 바로 이동하게 됩니다.
    - `함수형 에러 처리`: 예외가 값으로 표현되기 때문에 호출자는 예외를 명시적으로 처리하거나 전파할 수 있으며, 코드의 흐름이 끊기지 않습니다.
4. **가독성 및 유지보수 (Readability and Maintainability):**
    - `try/catch`: 예외 처리 코드가 코드의 중간에 섞여 있을 수 있어 가독성이 떨어질 수 있습니다.
    - `함수형 에러 처리`: 값을 통한 명시적인 에러 처리가 코드를 더 읽기 쉽게 만들 수 있습니다.

어떤 방식이 더 적절한지는 프로젝트의 특성, 개발자의 선호도, 코드베이스의 일관성 등에 따라 다를 수 있습니다. 함수형 에러 처리는 순수 함수와 결합하면서 코드를 더 안정적이고 예측 가능하게 만들 수 있습니다.

</aside>