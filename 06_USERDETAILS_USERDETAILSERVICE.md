# Spring Security — UserDetails & UserDetailsService

---

## Why These Even Exist

When someone hits your API, Spring Security stops them and asks:
```
→ Who are you?
→ Are you allowed here?
```

To answer this, Spring needs to find the user from YOUR database and understand their information.

Problem — Spring does not know:
```
→ What your User class looks like
→ How to find it in your database
```

Solution:
```
UserDetails        → teaches Spring what your User looks like
UserDetailsService → teaches Spring how to find your User in DB
```

---

## UserDetails

### Simple Definition
A contract (interface) you implement on your User class so Spring Security can understand it.

### Before vs After

```
Without UserDetails:
Spring → "I have no idea what a User is" 

With UserDetails:
Spring → "I can see username, password, roles — I understand this" 
```

### Code

```java
public class User implements UserDetails {

    private String username;
    private String password;
    private Set<Role> roles;

    @Override
    public String getUsername() { return username; }

    @Override
    public String getPassword() { return password; }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority(role.getRoleName().name()))
                .collect(Collectors.toList());
    }

    // Always return true — otherwise Spring rejects every login
    @Override public boolean isAccountNonExpired()     { return true; }
    @Override public boolean isAccountNonLocked()      { return true; }
    @Override public boolean isCredentialsNonExpired() { return true; }
    @Override public boolean isEnabled()               { return true; }
}
```

### What Each Method Does
| Method | Purpose |
|---|---|
| `getUsername()` | Spring reads this to identify the user |
| `getPassword()` | Spring reads this to verify password |
| `getAuthorities()` | Spring reads this to check roles |
| `isAccountNonLocked()` etc. | Must return `true` or Spring blocks every login |

---

## UserDetailsService

### Simple Definition
An interface with one job — given a username, find that user in DB and return it.

### Code

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    UserRepository userRepo;

    @Override
    public UserDetails loadUserByUsername(String username) {
        return userRepo.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }
}
```

### What Each Line Does

```java
@Service
// Spring finds and registers this class automatically

implements UserDetailsService
// Telling Spring: "come to me when you need to find a user"

loadUserByUsername(String username)
// Spring calls this automatically during every login

userRepo.findByUsername(username)
// goes to DB, finds the user

.orElseThrow(...)
// user not found → throw error → Postman gets 401 Unauthorized

return user
// works because User implements UserDetails 
```

---

## Order of Execution

```
Step 1 — UserDetailsService runs FIRST
→ Spring calls loadUserByUsername("Ali")
→ Goes to DB, finds Ali, returns him

Step 2 — UserDetails runs SECOND
→ Spring reads getUsername()
→ Spring reads getPassword()
→ Spring reads getAuthorities()

Step 3 — Spring Security runs THIRD
→ Verifies password 
→ Checks roles 
→ Allows or rejects the request
```

---

## Full Login Flow

```
POST /api/login { "username": "Ali", "password": "1234" }
        ↓
Spring Security intercepts
        ↓
Calls loadUserByUsername("Ali")         ← UserDetailsService
        ↓
Finds Ali in DB, returns him
        ↓
Spring reads Ali's password and roles   ← UserDetails
        ↓
Password matches   Roles checked 
        ↓
JWT token generated → sent to Postman
```

## Every Request After Login

```
GET /api/orders
Authorization: Bearer <token>
        ↓
Spring reads token → gets username "Ali"
        ↓
Calls loadUserByUsername("Ali") again   ← UserDetailsService
        ↓
Checks if Ali has permission            ← UserDetails (getAuthorities)
        ↓
Allowed  → returns orders
```

---

## One Line Each

```
UserDetails        → Form that teaches Spring what your User looks like
UserDetailsService → Person who fetches that form from the database
Spring Security    → Manager who reads the form and decides access
```

---

## Common Mistakes

| Mistake | What Happens |
|---|---|
| Not implementing `UserDetails` on User | Spring cannot understand your User object |
| Returning `false` on `isAccountNonLocked()` etc. | Spring rejects every login even with correct password |
| Forgetting `@Service` on UserDetailsServiceImpl | Spring cannot find your class, authentication breaks completely |
| Not throwing `UsernameNotFoundException` | Spring cannot handle the error properly |

---
