<script>
  import meetups from "../../meetups-store";
  import { createEventDispatcher } from "svelte";
  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/Button.svelte";
  import Modal from "../UI/Modal.svelte";
  import { isEmpty, isValidEmail } from "../../helpers/validation.js";

  export let id = null;

  let title = "";
  let subtitle = "";
  let address = "";
  let email = "";
  let description = "";
  let imageUrl = "";

  if (id) {
    const unsubscribe = meetups.subscribe(items => {
      const selectedMeetup = items.find(i => i.id === id);
      title = selectedMeetup.title;
      subtitle = selectedMeetup.subtitle;
      address = selectedMeetup.address;
      email = selectedMeetup.contactEmail;
      description = selectedMeetup.description;
      imageUrl = selectedMeetup.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: addressValid = !isEmpty(address);
  $: descriptionValid = !isEmpty(description);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: emailValid = isValidEmail(email);
  $: formIsValid =
    titleValid &&
    subtitleValid &&
    addressValid &&
    descriptionValid &&
    imageUrlValid &&
    emailValid;

  function submitForm() {
    const meetupData = {
      title: title,
      subtitle: subtitle,
      description: description,
      imageUrl: imageUrl,
      contactEmail: email,
      address: address
    };

    // meetups.push(newMeetup); // DOES NOT WORK!
    if (id) {
      fetch(`https://svelte-meetup-20dbb-default-rtdb.firebaseio.com/meetups/${id}.json`,{
        method: 'PATCH',
        body: JSON.stringify(meetupData),
        headers: { 'Content-type': 'application/json' }
      })
      .then(res => {
        if (!res.ok){
          throw new Error('An error occurred, please try again!');
        }
        meetups.updateMeetup(id, meetupData);
      })
      .catch(err => {
        console.log(err);
      });
    } else {
      fetch('https://svelte-meetup-20dbb-default-rtdb.firebaseio.com/meetups.json',{
        method: 'POST',
        body: JSON.stringify({...meetupData, isFavorite: false}),
        headers: { 'Content-type': 'application/json' }
      })
      .then(res => {
        if (!res.ok){
          throw new Error('An error occurred, please try again!');
        }
        return res.json();
      })
      .then(data => {
        meetups.addMeetup({
          ...meetupData, 
          isFavorite: false, 
          id: data.name
        });
      })
      .catch(err => {
        console.log(err);
      });

    }
    dispatch("save");
  }

  function deleteMeetup() {
    fetch(`https://svelte-meetup-20dbb-default-rtdb.firebaseio.com/meetups/${id}.json`,{
        method: 'DELETE',
    })
    .then(res => {
      if (!res.ok){
        throw new Error('An error occurred, please try again!');
      }
      meetups.removeMeetup(id);
    })
    .catch(err => {
      console.log(err);
    });
    dispatch("save");
  }

  function cancel() {
    dispatch("cancel");
  }
</script>

<style>
  form {
    width: 100%;
  }
</style>

<Modal title="?????? ??????" on:cancel>
  <form on:submit={submitForm}>
    <TextInput
      id="title"
      label="??????"
      valid={titleValid}
      validityMessage="????????? ????????? ?????????."
      value={title}
      on:input={event => (title = event.target.value)} />
    <TextInput
      id="subtitle"
      label="?????????"
      valid={subtitleValid}
      validityMessage="???????????? ????????? ?????????."
      value={subtitle}
      on:input={event => (subtitle = event.target.value)} />
    <TextInput
      id="address"
      label="??????"
      valid={addressValid}
      validityMessage="????????? ????????? ?????????."
      value={address}
      on:input={event => (address = event.target.value)} />
    <TextInput
      id="imageUrl"
      label="????????? URL"
      valid={imageUrlValid}
      validityMessage="????????? url??? ????????? ?????????"
      value={imageUrl}
      on:input={event => (imageUrl = event.target.value)} />
    <TextInput
      id="email"
      label="E-Mail"
      type="email"
      valid={emailValid}
      validityMessage="????????? ????????? ????????? ?????????."
      value={email}
      on:input={event => (email = event.target.value)} />
    <TextInput
      id="description"
      label="????????????"
      controlType="textarea"
      valid={descriptionValid}
      validityMessage="????????? ????????? ????????? ?????????."
      bind:value={description} />
  </form>
  <div slot="footer">
    <Button type="button" mode="outline" on:click={cancel}>??????</Button>
    <Button type="button" on:click={submitForm} disabled={!formIsValid}>
      ??????
    </Button>
    {#if id}
    <Button type="button" on:click={deleteMeetup}>??????</Button>
    {/if}
  </div>
</Modal>
