## Usage

accepts a csv file that must have this below as the header:

```
offering_no,course_no,class_type,room,days,time,instructor,section,no_of_hours,remark
```

### CSV format

`day`: concatenated string of any of the following: "M", "T", "W", "Th", "F"

`time`: string of a range of time period, with the anything in this range written like "07:00-19:00"

`remark`: number, turns reddish if value is > 0, grayish otherwise

Almost everything else is used as a search filter

# sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
