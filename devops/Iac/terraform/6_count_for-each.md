# Terraform - Count & for_each

**Ref**

* count : https://www.terraform.io/docs/language/meta-arguments/count.html
* for_each : https://www.terraform.io/docs/language/meta-arguments/for_each.html

## Provider

```json
provider "aws" {
  region = "ap-northeast-2"
}
```

## No Count

```json
/*
 * No count / for_each
 */
resource "aws_iam_user" "user_1" {
  name = "user-1"
}

resource "aws_iam_user" "user_2" {
  name = "user-2"
}

resource "aws_iam_user" "user_3" {
  name = "user-3"
}

output "user_arns" {
  value = [
    aws_iam_user.user_1.arn,
    aws_iam_user.user_2.arn,
    aws_iam_user.user_3.arn,
  ]
}
```

## Count

```json
/*
 * count
 */

resource "aws_iam_user" "count" {
  count = 10

  name = "count-user-${count.index}"
}

output "count_user_arns" {
  value = aws_iam_user.count.*.arn
}
```

* resource, data와 module에서 적용이 가능
* resource type과 관계없이 지원해주는 속성을 **meta-argument**라고 한다.
* count meta-argument는 resource body 최상단에다가 정의한다.
* count.index로 count의 숫자의 접근 가능

### count 단점

count로 user를 생성했다. 그 후 직원이 퇴사할 경우 user를 삭제해야 하는데, 중간 인덱스인 경우 인덱스가 하나씩 올라간다. 그렇게 되면 테라폼 상에서 해당 유저 밑으로 삭제되고 재생성되는 **issue**가 생긴다. 이것을 해결하기 위해서 수동적으로 관리해야 하는 cost가 발생하는데, for_each를 사용하면 hash 형태이기 때문에 관리가 수월해진다.



## for_each

```json
/*
 * for_each
 */

resource "aws_iam_user" "for_each_set" {
  for_each = toset([
    "for-each-set-user-1",
    "for-each-set-user-2",
    "for-each-set-user-3",
  ])

  name = each.key
}

output "for_each_set_user_arns" {
  value = values(aws_iam_user.for_each_set).*.arn
}

resource "aws_iam_user" "for_each_map" {
  for_each = {
    alice = {
      level = "low"
      manager = "posquit0"
    }
    bob = {
      level = "mid"
      manager = "posquit0"
    }
    john = {
      level = "high"
      manager = "steve"
    }
  }

  name = each.key

  tags = each.value
}

output "for_each_map_user_arns" {
  value = values(aws_iam_user.for_each_map).*.arn
}
```

* resource, data와 module에서 적용이 가능
* for_each meta-argument는 resource body 최상단에다가 정의한다.
* set
  * set : List이지만 unique element를 출력
  * list를 `toset`을 이용해서 형변환 함
  * `each.key`와` each.value`로 접근 가능. 하지만 set은 k-v형식이 아니여서 서로가 동일한 값을 출력
* map
  * map : { k1 : v1, k2 : v2 } 형식
  * `each.key`와` each.value`로 접근 가능
  * key 값은 string이어야 함
  * value 값은 type 상관 없음