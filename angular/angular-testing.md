[Back](/my-notes/angular/overview)

# Testing In Angular

_It may be helpful to grab my repo of completed exercises from a course (Unit Testing In Angular by Joe Eames on Pluralsight): [seanmccay/angular-testing](https://github.com/seanmccay/angular-testing) so that you can see the components that are being tested_

### 3 types of tests in Angular:

1. Unit tests
    - Will be the most common and useful
    - Should be flexible and harder to break
    - Test only the component class and it's behavior, not it's template
    - Doesn't require setting up a test bed/module
2. Shallow Integration tests
    - Test the component and its template
    - Need to create a test bed/module and mock all services/imports/dependencies
    - Mock children/ignore them
3. Deep Integration tests
    - Should be used sparingly
    - They can be brittle
    - Tests the component and its interactions with its children/dependencies
    - Render out all of the components children
    - Use live children components, requires a lot more setup
    - End to end might be considered integration testing

Make use of the browser console when using Karma, it can sometimes give you information that karma/jasmine errors don't

Run `ng test --source-map=false` which helps get past some problems with zoneJS (recommended by Joe Eames)

Make sure that you only test your code, not testing the framework. For instance when testing a routing component, don't test to make sure the actual routing works because we know that Angular Router works.

When building tests, refrain using `NO_ERRORS_SCHEMA` unless absolutely necessary as it can hide errors that you could easily fix like misspellings in the template or something wrong when building a form. Usually you can solve issues by importing modules that are needed into the TestBed or mocking your dependencies, router, etc.

Use shallow/deep integration tests only when you need to dig into the template and make sure that your template is working with your component correctly.

## Unit tests

```js
describe('Heroes Component (unit tests)', () => {
	let component: HeroesComponent;
	let HEROES;
	let mockHeroService;

	beforeEach(() => {
		HEROES = [
			{ id: 1, name: 'testhero1', strength: 5 },
			{ id: 2, name: 'testhero2', strength: 8 },
			{ id: 3, name: 'testhero3', strength: 10 }
		];

		mockHeroService = jasmine.createSpyObj([
			'getHeroes',
			'addHero',
			'deleteHero'
		]);

		component = new HeroesComponent(mockHeroService);
	});

	describe('delete', () => {
		// test state
		it('should remove indicated hero from array', () => {
			mockHeroService.deleteHero.and.returnValue(of(true));
			component.heroes = HEROES;

			component.delete(HEROES[2]);

			expect(component.heroes.length).toBe(2);
			expect(
				component.heroes.map(h => h.name).includes('testhero3')
			).toBe(false);
		});

		// an interaction test
		it('should call heroService.deleteHero with the correct hero', () => {
			mockHeroService.deleteHero.and.returnValue(of(true));
			component.heroes = HEROES;

			component.delete(HEROES[2]);

			expect(mockHeroService.deleteHero).toHaveBeenCalledWith(HEROES[2]);
		});

		it('should subscribe to the observable of heroService.deleteHero', () => {});
	});
});
```

## Shallow Tests

Tests for component with no child components

```js
describe('Hero Component (shallow tests)', () => {
	let fixture: ComponentFixture<HeroComponent>;

	beforeEach(() => {
		TestBed.configureTestingModule({
			imports: [],
			declarations: [HeroComponent]
		});
		fixture = TestBed.createComponent(HeroComponent);
	});

	it('should have the correct hero', () => {
		fixture.componentInstance.hero = {
			id: 1,
			name: 'testhero',
			strength: 8
		};
		expect(fixture.componentInstance.hero.name).toEqual('testhero');
	});

	it('should render the hero name in an anchor tag', () => {
		fixture.componentInstance.hero = {
			id: 1,
			name: 'testhero',
			strength: 8
		};

		// * run change detection / init component
		fixture.detectChanges();

		// you can use the debug element or native element. debug element has a few more capabiliites available.
		expect(
			fixture.debugElement.query(By.css('a')).nativeElement.textContent
		).toContain('testhero');

		expect(fixture.nativeElement.querySelector('a').textContent).toContain(
			'testhero'
		);
	});
});
```

Tests for component with child component(s)

```js
describe('Heroes Component (shallow tests)', () => {
	let fixture: ComponentFixture<HeroesComponent>;
	let mockHeroService;
	let HEROES;

	// mock child component
	@Component({
		selector: 'app-hero',
		template: '<div></div>'
	})
	class FakeHeroComponent {
		@Input() hero: Hero;
		// @Output() delete = new EventEmitter();

		onDeleteClick($event): void {
			$event.stopPropagation();
			this.delete.next();
		}
	}

	beforeEach(() => {
		HEROES = [
			{ id: 1, name: 'testhero1', strength: 5 },
			{ id: 2, name: 'testhero2', strength: 8 },
			{ id: 3, name: 'testhero3', strength: 10 }
		];

		mockHeroService = jasmine.createSpyObj([
			'getHeroes',
			'addHero',
			'deleteHero'
		]);

		TestBed.configureTestingModule({
			declarations: [HeroesComponent, FakeHeroComponent],
			providers: [{ provide: HeroService, useValue: mockHeroService }]
			// can use to ignore child elements in the template, but try to avoid this. Just mock the children.
			// schemas: [NO_ERRORS_SCHEMA]
		});
		fixture = TestBed.createComponent(HeroesComponent);
	});

	it('should set heroes correctly from service', () => {
		mockHeroService.getHeroes.and.returnValue(of(HEROES));
		fixture.detectChanges();

		expect(fixture.componentInstance.heroes.length).toBe(3);
	});

	it('should render one li for each hero', () => {
		mockHeroService.getHeroes.and.returnValue(of(HEROES));
		fixture.detectChanges();

		expect(fixture.nativeElement.querySelectorAll('li').length).toBe(
			HEROES.length
		);
	});
});
```

## Deep Tests

```js
@Directive({
	// tslint:disable-next-line: directive-selector
	selector: '[routerLink]'
})
class RouterLinkDirectiveStub {
	@Input('routerLink') linkParams: any;
	navigatedTo: any = null;

	@HostListener('click') onClick() {
		this.navigatedTo = this.linkParams;
	}
}

describe('Heroes Component (deep tests)', () => {
	let fixture: ComponentFixture<HeroesComponent>;
	let HEROES: Hero[];
	let mockHeroService;

	beforeEach(() => {
		HEROES = [
			{ id: 1, name: 'PiderMan', strength: 8 },
			{ id: 2, name: 'DuperMan', strength: 10 },
			{ id: 2, name: 'BadMan', strength: 5 }
		];

		mockHeroService = jasmine.createSpyObj([
			'getHeroes',
			'addHero',
			'deleteHero'
		]);

		TestBed.configureTestingModule({
			declarations: [
				HeroesComponent,
				HeroComponent,
				RouterLinkDirectiveStub
			],
			providers: [{ provide: HeroService, useValue: mockHeroService }]
		});

		fixture = TestBed.createComponent(HeroesComponent);
	});

	it('should should render each hero as hero component', () => {
		mockHeroService.getHeroes.and.returnValue(of(HEROES));
		// when running change detection on parent, it gets run on all children
		fixture.detectChanges();

		// one of the cool capabilities of the debugElement
		const heroComponents = fixture.debugElement.queryAll(
			// component is a subclass of Directive in Angular, so you can use By.directive to find component Directives
			// as well as attribute directives
			By.directive(HeroComponent)
		);

		// with debug elements you can also get a hook into the componentInstance as well
		expect(heroComponents.length).toEqual(HEROES.length);
		for (let i = 0; i < heroComponents.length; i++) {
			// toEqual is a deep equivalency, so it checks props
			expect(heroComponents[i].componentInstance.hero).toEqual(HEROES[i]);
		}
	});

	it(`should call heroService.deleteHero when the hero
      component's delete button is clicked`, () => {
		// watch HeroesComponent.delete to see if it gets invoked
		spyOn(fixture.componentInstance, 'delete');

		mockHeroService.getHeroes.and.returnValue(of(HEROES));
		fixture.detectChanges();

		const heroComponents = fixture.debugElement.queryAll(
			By.directive(HeroComponent)
		);

		// * you can trigger the HTML click, or you could just tell the child component to raise its event.
		// * Here we'll just raise the event because the HTML click event handling is more specific to
		// * the child component and should be tested in its own integration tests

		// heroComponents[0]
		//   .query(By.css("button"))
		//   .triggerEventHandler("click", { stopPropagation: () => {} });

		// * When triggering with event, we can get an instance of the component, or we could
		// * grab the debugElement which has a nice tool to trigger events as well
		// (<HeroComponent>heroComponents[0].componentInstance).delete.emit(undefined);
		heroComponents[0].triggerEventHandler('delete', undefined);

		expect(fixture.componentInstance.delete).toHaveBeenCalledWith(
			HEROES[0]
		);
	});

	it('should add a new hero to the hero list when the add button is clicked', () => {
		mockHeroService.getHeroes.and.returnValue(of(HEROES));
		fixture.detectChanges();

		const name = 'Hero Bro';
		mockHeroService.addHero.and.returnValue(
			of({ id: 5, name: name, strength: 5 })
		);

		const inputElement = fixture.debugElement.query(By.css('input'))
			.nativeElement;
		const addButton = fixture.debugElement.queryAll(By.css('add'))[0]
			.nativeElement;

		// simulate typing into input and clicking add button
		inputElement.value = name;
		addButton.triggerEventHandler('click', null);

		// have to run change detection in order to update template
		fixture.detectChanges();

		const heroText = fixture.debugElement.query(By.css('ul')).nativeElement
			.textContent;
		expect(heroText).toContain('Hero Bro');
	});

	it('should have the correct route for the first hero', () => {
		const heroComponents = fixture.debugElement.queryAll(
			By.directive(HeroComponent)
		);

		const routerLink =
			heroComponents[0].query(By.directive(RouterLinkDirectiveStub))
				.injector.get <
			RouterLinkDirectiveStub >
			RouterLinkDirectiveStub;

		heroComponents[0].query(By.css('a')).triggerEventHandler('click', null);

		expect(routerLink.navigatedTo).toBe('/detail/1');
	});
});
```

## Mocking HTTP with Service Tests

```js
describe('Hero Service (integration tests)', () => {
	let mockMessageService;
	let httpTestingController: HttpTestingController;
	let service;

	beforeEach(() => {
		mockMessageService = jasmine.createSpyObj(['add']);
		TestBed.configureTestingModule({
			imports: [HttpClientTestingModule],
			providers: [
				HeroService,
				{ provide: MessageService, useValue: mockMessageService }
			]
		});

		httpTestingController = TestBed.get(HttpTestingController);
		service = TestBed.get(HeroService);
	});

	describe('getHero', () => {
		it('should call get with the correct url', () => {
			service.getHero(4).subscribe(() => {
				console.log('does not execute until the request is flushed');
			});

			const request = httpTestingController.expectOne('api/heroes/4');
			request.flush({ id: 4, name: 'testhero', strength: 100 });
			httpTestingController.verify();
		});
	});
});
```
