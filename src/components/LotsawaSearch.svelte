<script>
  import axios from "axios";
  import cheerio from "cheerio";
  import Select from "svelte-select";
  import Flag from "./Flag.svelte";
  import ResultCard from "./ResultCard.svelte";
  import { fade } from "svelte/transition";
  import { writable } from "svelte/store";
  import "flag-icons/css/flag-icons.min.css";

  let pageLoaded = false;
  let loading = false;
  let errors = false;
  let prayersWithMatchingLines = [];

  const searchTerm = writable("རང་རིག་");
  const language = writable("en");

  if (typeof window !== "undefined") {
    setTimeout(() => {
      pageLoaded = true;
    }, 400);
    const storedSearchTerm = localStorage.getItem("searchTerm");
    const storedLanguage = localStorage.getItem("language");

    if (storedSearchTerm) searchTerm.set(storedSearchTerm);
    if (storedLanguage) language.set(storedLanguage);

    searchTerm.subscribe((value) => {
      loading = false;
      prayersWithMatchingLines = [];
      localStorage.setItem("searchTerm", value);
    });

    language.subscribe((value) => {
      loading = false;
      prayersWithMatchingLines = [];
      localStorage.setItem("language", value);
    });
  }

  let languages = [
    { value: "en", countryCode: "us" },
    { value: "de", countryCode: "de" },
    { value: "es", countryCode: "es" },
    { value: "fr", countryCode: "fr" },
    { value: "it", countryCode: "it" },
    { value: "nl", countryCode: "nl" },
    { value: "pt", countryCode: "pt" },
    { value: "zh", countryCode: "cn" },
  ];

  function marker(text, regexp) {
    const markTerm = (term) =>
      `<mark class="bg-transparent underline">${term}</mark>`;
    return text.replace(regexp, (term) => markTerm(term));
  }

  $: loadPrayers = function () {
    loading = true;
    prayersWithMatchingLines = [];
    let url = encodeURIComponent(
      `https://www.lotsawahouse.org/Cgi/search.pl?searchlang=${$language}&op=Search&q="${$searchTerm}"`
    );
    axios
      .get(
        `https://workers-playground-damp-lake-fba2.frere-jeremy.workers.dev/?${url}`
      )
      .catch(function (response) {
        console.log("ERROR:", response);
        errors = true;
      })
      .then(function (response) {
        let normalize = (text) => text.replace(/ /g, " ");
        let listPage = cheerio(response.data);
        let prayers = listPage
          .find(".search-result")
          .map(function (_, searchResult) {
            let path = listPage.find(searchResult).find("a").attr("href");
            return {
              name: listPage
                .find(searchResult)
                .find("p:first")
                .text()
                .replace(/\n\t/g, "")
                .replace(/[ ]+/g, " ")
                .trim(),
              url: `https://www.lotsawahouse.org${path}`,
            };
          });
        let promises = [];
        prayers
          .filter(function (_, prayer) {
            if ($language == "en") {
              return prayer.url.match(
                /^https:\/\/www.lotsawahouse.org\/[a-z]{3,}.*/
              );
            } else {
              return prayer.url.match(
                `^https:\/\/www.lotsawahouse.org\/${$language}\/`
              );
            }
          })
          .each(function (_, prayer) {
            let promise = axios
              .get(
                `https://workers-playground-damp-lake-fba2.frere-jeremy.workers.dev/?${encodeURIComponent(
                  prayer.url
                )}`
              )
              .then(function (response) {
                if (response.data.length < 256)
                  // Avoids redirects
                  return;
                let prayerPage = cheerio(response.data);
                let lines = [];
                prayerPage
                  .find(".HeadingTib")
                  .filter((_, tibetanTitle) =>
                    normalize(cheerio(tibetanTitle).text()).includes(
                      normalize($searchTerm)
                    )
                  )
                  .each((_, tibetanTitle) => {
                    lines.push({
                      tibetan: marker(
                        cheerio(tibetanTitle).text(),
                        new RegExp($searchTerm, "g")
                      ),
                      french: cheerio(tibetanTitle).next().text(),
                      title: true,
                    });
                  });
                prayerPage
                  .find(".TibetanVerse")
                  .filter((_, tibetanVerse) =>
                    normalize(cheerio(tibetanVerse).text()).includes(
                      normalize($searchTerm)
                    )
                  )
                  .each((_, tibetanVerse) => {
                    lines.push({
                      tibetan: marker(
                        cheerio(tibetanVerse).text(),
                        new RegExp($searchTerm, "g")
                      ),
                      french: cheerio(tibetanVerse).next().next().text(),
                      previousTibetan: cheerio(tibetanVerse)
                        .prevAll(".TibetanVerse:first")
                        .text(),
                      previousFrench: cheerio(tibetanVerse)
                        .prevAll(".EnglishText:first")
                        .text(),
                      nextTibetan: cheerio(tibetanVerse)
                        .nextAll(".TibetanVerse:first")
                        .text(),
                      nextFrench: cheerio(tibetanVerse)
                        .nextAll(".TibetanVerse:first")
                        .nextAll(".EnglishText:first")
                        .text(),
                    });
                  });
                prayerPage
                  .find(".TibetanExplanation")
                  .filter((_, tibetanSmallWritings) =>
                    normalize(cheerio(tibetanSmallWritings).text()).includes(
                      normalize($searchTerm)
                    )
                  )
                  .each((_, tibetanSmallWritings) => {
                    lines.push({
                      tibetan: marker(
                        cheerio(tibetanSmallWritings).text(),
                        new RegExp($searchTerm, "g")
                      ),
                      french: cheerio(tibetanSmallWritings).next().text(),
                      smallLetters: true,
                    });
                  });
                if (lines.length) {
                  prayersWithMatchingLines.push({
                    name: prayer.name,
                    url: prayer.url,
                    lines: lines,
                  });
                  prayersWithMatchingLines = prayersWithMatchingLines; // Trigger reactivity
                }
              });
            promises.push(promise);
          });
        Promise.all(promises).then(function () {
          loading = false;
        });
      })
      .catch(function (error) {
        console.log(error);
      })
      .finally(function () {});
  };
</script>

{#if pageLoaded}
  <div transition:fade>
    <h1 class="flex items-center font-serif text-3xl mb-8 h-[64px]">
      <img
        src="/lotsawa-search/favicon.svg"
        alt="Logo"
        class="rounded mr-4 w-[64px]"
      />
      Lotsawa Search
    </h1>
    <div class="flex flex-col gap-4 items-stretch justify-between">
      <form action="#">
        <label
          for="default-search"
          class="mb-2 text-sm font-medium text-gray-900 sr-only">Search</label
        >
        <div class="relative">
          <div
            class="absolute inset-y-0 left-1 flex items-center"
            style="border-right: 1px solid lightgray"
          >
            <Select
              items={languages}
              value={$language}
              bind:justValue={$language}
              searchable={false}
              clearable={false}
              --width="64px"
              --border="none"
              --border-hover="none"
              --border-focused="none"
            >
              <div slot="item" let:item let:index>
                <Flag {item} />
              </div>
              <div class="select-selection" slot="selection" let:selection>
                <Flag item={selection} />
              </div>
            </Select>
          </div>
          <input
            bind:value={$searchTerm}
            autofocus
            type="search"
            id="search-input"
            class="block w-full text-gray-900 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-700 focus:border-red-700 outline-none"
            placeholder="Tibetan expression"
            required
          />
          <button
            on:click={loadPrayers}
            type="submit"
            class="text-white w-[100px] flex justify-center absolute right-2.5 bottom-2.5 cursor-pointer bg-red-700 hover:bg-red-800 focus:ring-2 focus:outline-none focus:ring-red-300 font-medium rounded-lg text-sm px-4 py-2"
            disabled={loading}
          >
            {#if loading}
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="18"
                height="18"
                viewBox="0 0 24 24"
                ><path
                  fill="currentColor"
                  d="M12,4a8,8,0,0,1,7.89,6.7A1.53,1.53,0,0,0,21.38,12h0a1.5,1.5,0,0,0,1.48-1.75,11,11,0,0,0-21.72,0A1.5,1.5,0,0,0,2.62,12h0a1.53,1.53,0,0,0,1.49-1.3A8,8,0,0,1,12,4Z"
                  ><animateTransform
                    attributeName="transform"
                    dur="0.75s"
                    repeatCount="indefinite"
                    type="rotate"
                    values="0 12 12;360 12 12"
                  /></path
                ></svg
              >
            {:else}
              Search
            {/if}
          </button>
        </div>
      </form>

      {#if errors}
        <div
          class="flex items-center gap-4 rounded-lg leading-relaxed p-4 pl-5 bg-red-100 border border-gray-300"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="64"
            height="64"
            viewBox="0 0 16 16"
            class="text-red-700"
            ><path
              fill="currentColor"
              d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0zM8 4a.905.905 0 0 0-.9.995l.35 3.507a.552.552 0 0 0 1.1 0l.35-3.507A.905.905 0 0 0 8 4zm.002 6a1 1 0 1 0 0 2a1 1 0 0 0 0-2z"
            /></svg
          >
          <div>Boom. Something didn't go as planned.</div>
        </div>
      {/if}

      {#each prayersWithMatchingLines as prayer (prayer.url)}
        <ResultCard {prayer} />
      {/each}
    </div>
  </div>
{/if}

<style>
  .select-selection {
    margin-left: 4px;
  }
  #search-input {
    padding: 9px 118px 9px 78px;
    font-family: "DDC Uchen";
    font-size: 18px;
    line-height: 2em;
    color: #954a29;
  }
</style>
