export default {
 async fetch(request,env){
  const u=new URL(request.url), p=u.pathname;
  if(p==="/api/titles"&&request.method==="POST"){
   if(!env.LUPIN_TITLES)return J({error:"KV_NOT_BOUND"},503);
   const b=await request.json().catch(()=>null), title=b?.title?.trim(), image=typeof b?.image==="string"?b.image:null;
   if(!title)return J({error:"INVALID_TITLE"},400);
   if(image&&image.length>6_000_000)return J({error:"IMAGE_TOO_LARGE"},413);
   const id=crypto.randomUUID().replaceAll("-","").slice(0,7);
   await env.LUPIN_TITLES.put(id,JSON.stringify({title,image,created:Date.now()})); return J({id,title});
  }
  const m=p.match(/^\/api\/titles\/([A-Za-z0-9_-]+)$/);
  if(m){
   if(!env.LUPIN_TITLES)return J({error:"KV_NOT_BOUND"},503);
   if(request.method==="GET"){const v=await env.LUPIN_TITLES.get(m[1]);return v?new Response(v,{headers:{"content-type":"application/json"}}):J({error:"NOT_FOUND"},404)}
   if(request.method==="PUT"){
    const b=await request.json().catch(()=>null),title=b?.title?.trim();if(!title)return J({error:"INVALID_TITLE"},400);
    const old=await env.LUPIN_TITLES.get(m[1],"json");if(!old)return J({error:"NOT_FOUND"},404);
    await env.LUPIN_TITLES.put(m[1],JSON.stringify({...old,title,updated:Date.now()}));return J({id:m[1],title})
   }
   if(request.method==="DELETE"){await env.LUPIN_TITLES.delete(m[1]);return J({ok:true})}
  }
  const s=p.match(/^\/t\/([A-Za-z0-9_-]+)$/); if(s){const x=new URL("/play.html",u);x.searchParams.set("id",s[1]);if(u.searchParams.get("admin")==="1")x.searchParams.set("admin","1");return Response.redirect(x,302)}
  return env.ASSETS.fetch(request);
 }
}; const J=(x,s=200)=>new Response(JSON.stringify(x),{status:s,headers:{"content-type":"application/json"}});
