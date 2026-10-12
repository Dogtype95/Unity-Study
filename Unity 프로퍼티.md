
- SerializeField
```cs 
// public 변수: 인스펙터에 노출되지만, 다른 클래스에서도 접근 가능 (캡슐화 훼손) 
public float moveSpeed = 5.0f;
 // private + [SerializeField]: 인스펙터에 노출되면서도 외부 클래스 접근 차단 (권장) 
 [SerializeField] private int maxHealth = 100;

```
- 