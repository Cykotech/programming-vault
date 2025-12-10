# Client Side Auth

**Date:** 11/29/25  
**Current Phase:** MVP

---
#react #auth #ts #contextAPI 
## What I Worked On

I worked on enabling my front end client being able to login into the application using a 4 digit pin.

## What Changed Since Last Session

A new service was initialized to handle all authentication requests. Context API was used to pass the token retrieved from the server to the entire application. Routes were also moved from `main.tsx` to a separate wrapper for the application. 

## Problems I Hit

The main issue that needed to be tackled was implementing Context API to handle the token.

## Technical Notes

Initializing the state of the authentication begins with creating an instance of the ContextAPI and if type associated with that context is an object, creating an interface to supply. If there is no default value to supply, we can provide `null`.

```ts
export const AuthContext = createContext<IAuthContext>(null!);

interface IAuthContext {
	token: string | null;
	setToken:(t: string | null) => void;
}
```

A React component is then created to wrap around the portion of the app that needs to be protected. This will include the state variable and setter, additional logic for validation, the context value to assign the provider, and provider itself.

```tsx
export function AuthProvider({children}) {
	// The state setter is not exposed directly to prevent accidental
	// changes to the token
	const [token, setToken_] = useState(localStorage.getItem("token"));
	
	const setToken = (newToken: string | null) => {
		setToken_(newToken);
	}
	
	// A useEffect is used to update the headers whenever the token is
	// updated or deleted
	useEffect(() => {
		if (token) {
			axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
			localStorage.setItem("token", token); 
		} else {
			delete axios.defaults.headers.common['Authorization'];
			localStorage.removeItem("token");
		}
	}, [token]);
	
	// The token value is then memoized so the context only updates when the
	// state changes
	const contextValue = useMemo<IAuthContext>(() => ({token, setToken}),
		[token]);
	
	// The provider is then returned with the memoized value supplied to it
	return (
		<AuthContext.Provider value={contextValue}>
			{children}
		</AuthContext.Provider>
	);
}
```

A custom hook is then created so that the context and context hook are not exposed directly.

```ts
export function useAuth() {
	return useContext(AuthContext);
}
```

In order to provide the authentication to the application, the provider will then need to be wrapped around the portion of the app that requires authentication.

```tsx
function App() {
	return(
		<AuthProvider>
			<RouteProvider/>
		</AuthProvider>
	)
}
```

Using and updating the authentication context is now as simple as calling the custom hook.

```tsx
// Login.tsx
export default function Login() {
	const {setToken} = useAuth();
	const [pin, setPin] = useState<string>("");
	
	return (
		<>
			<input type={"text"} onChange={e => setPin(e.target.value)}
				value={pin}/>  
			<p>Pin</p>  
		    <button type={"submit"} onClick={async () => {  
		        const response = await authService.login(pin);  
		        setToken(response);  
		    }}>Login</button>  
		</>
	)
}

// Routes.tsx
export function RouteProvider() {
	const {token} = useAuth();
	
	if (!token)
		return (<Login/>);
	
	// Rest of routes
}
```
## Decisions Made Today

Routes were extracted from `main.tsx` into their own component to provide an easier interface to wrap `AuthProvider` around. This will then enable any modifications if there is any desire for unprotected routes.

## Next Steps

Streamline the login process in the UI so that after a successful login, the login component still does not display. Provide a logout function that invalidates the token.

## Blockers to Watch

As with any authentication, always be vigilant for security vulnerabilities. Local storage might not be ideal for storing a JWT.

## References / Links

https://dev.to/sanjayttg/jwt-authentication-in-react-with-react-router-1d03
https://react.dev/reference/react/createContext#

---

## Session Summary

This project now provides a simple interface for users to login and interact with the system. Users are now able to create new tables under their names.
